# Nextcloud WebDAV Integration for Browser-Based Apps

A technical guide for connecting a static web application (running on any origin, e.g. GitHub Pages or a custom domain) to a Nextcloud instance over WebDAV, using the official **WebAppPassword** Nextcloud app.

---

## 1. Overview

A web page can only call another origin if that origin explicitly allows it via CORS. Browsers block cross-origin requests to Nextcloud because, since Nextcloud 34, the former `cors.allowed-domains` configuration option no longer exists and Nextcloud itself sends no CORS headers for WebDAV.

The **WebAppPassword** app (by digital blueprint, official Nextcloud App Store) solves this with two mechanisms:

1. **CORS headers** for the WebDAV/CalDAV endpoints (and optional OCS Share + Preview APIs), emitted for a configurable allowlist of origins — including the preflight `OPTIONS` handling browsers require.
2. **Temporary app passwords** issued to the browser, so the page can authenticate as a logged-in Nextcloud user without ever handling the account password.

Everything stays client-side: the Nextcloud credentials never leave the browser profile, and no server configuration beyond installing the app and listing allowed origins is required.

---

## 2. Server-Side Setup

### 2.1 Install the app

- From the Nextcloud web UI: **Apps → Security → WebAppPassword → Install**.
- Or via occ:

```bash
occ app:install webapppassword
```

Supported Nextcloud versions: 22 through 34 (check the App Store page for the current release). Custom apps are installed into the `custom_apps` directory, which survives Nextcloud updates in standard installations.

### 2.2 Configure allowed origins

In the admin settings page of the app (**Administration settings → WebAppPassword**), set the allowed origins for WebDAV/CalDAV (and separately for the files sharing API and preview API).

Rules:

- Origins are comma-separated, full origins with scheme, host and optional port (e.g. `https://app.example.com`). Paths are ignored.
- One-level subdomain wildcards are supported: `https://*.example.com` matches `https://app.example.com` but **not** `https://example.com` and **not** `https://a.b.example.com`.
- The apex domain must be listed explicitly if it should be allowed.
- Scheme and port must match exactly: `https://*.example.com` does not allow `http://app.example.com`.

The same allowlist gates both the CORS headers and the temporary-password issuance.

### 2.3 Config fallback

If the settings page values are empty, `config/config.php` keys are used:

```php
'webapppassword.origins' => ['https://app.example.com'],                 // WebDAV/CalDAV
'webapppassword.files_sharing_origins' => ['https://app.example.com'],   // Share API
'webapppassword.preview_origins' => ['https://app.example.com'],         // Preview API
```

---

## 3. Authentication Flow (Endpoint Contract)

The page obtains a temporary app password through a popup/tab flow:

1. **Open the login window** (must be triggered by a user gesture — popup blockers otherwise):

```
GET {server}/index.php/apps/webapppassword?target-origin={pageUrlEncoded}
```

   The `target-origin` query parameter must be the URL of the page that will receive the message (it is validated against the configured origins; a disallowed origin receives HTTP 403).

2. **The user logs in** in that window (or has an existing session).

3. **The app page requests a temporary token** (same-origin POST from the popup, includes the CSRF token):

```
POST {server}/index.php/apps/webapppassword/create
Headers: target-origin: {pageUrl}, requesttoken: {CSRF token}
```

   Response (JSON):

```json
{
  "loginName": "user",
  "token": "72-random-characters",
  "webdavUrl": "https://{server}/remote.php/dav/files/{uid}"
}
```

   The `token` is a temporary app password. It expires (cleaned up by a background job), so a `401` on a later request means the token is stale and the flow must be repeated.

4. **Delivery to the opener** — the popup script sends the result:

```js
window.opener.postMessage(
  { type: "webapppassword", loginName, token, webdavUrl },
  targetOrigin
);
```

5. **The page stores** the credentials client-side only and closes the window (script-opened windows may be closed programmatically).

> **Always verify the message sender:** in the receiving `message` listener, check `event.origin` against the Nextcloud server origin before accepting the data.

---

## 4. API Surface

### 4.1 WebDAV

- **Base URL:** use the server-provided `webdavUrl` verbatim (it contains the real user ID; do not derive it from `loginName`, which may differ). Default shape: `{server}/remote.php/dav/files/{uid}`.
- **Authentication header** on every request:

```
Authorization: Basic base64(loginName:token)
```

- **CORS:** the app answers the browser's preflight `OPTIONS` itself (HTTP 204 with `Access-Control-Allow-Origin` echoed for allowed origins) and adds the CORS headers to all DAV responses. No other server configuration is required.

Common operations:

| Operation | Method | Notes |
|---|---|---|
| List directory | `PROPFIND` | `Depth: 1` (or `0` for a single node); response is XML — parse `<response><href>` entries, distinguish folders via `<resourcetype><collection>` |
| Upload file | `PUT` | `Content-Type` as needed; body = file bytes |
| Download file | `GET` | |
| Create folder | `MKCOL` | Nested paths must be created segment by segment (a parent missing for `a/b` fails with 409/404) |
| Delete | `DELETE` | |

### 4.2 OCS Share API (optional)

CRUD over a CORS-enabled subset of the sharing API:

```
{server}/index.php/apps/webapppassword/api/v1/shares
```

Methods: `GET` (list, inherited, pending), `POST` (create), `PUT`/`DELETE` per share id; preflight `OPTIONS` is handled by the app. Controlled by `webapppassword.files_sharing_origins`.

### 4.3 Preview API (optional)

```
{server}/index.php/apps/webapppassword/core/preview
```

`GET` by file id plus preflight handling. Controlled by `webapppassword.preview_origins`.

---

## 5. Client-Side Reference

Minimal pattern (plain JavaScript):

```js
const server = "https://nextcloud.example.com";
let auth = null; // { loginName, token, webdavUrl }

function connect() {
  const url = server + "/index.php/apps/webapppassword?target-origin=" +
    encodeURIComponent(window.location.href);
  window.open(url, "_blank");
}

window.addEventListener("message", (event) => {
  const data = event.data;
  if (!data || data.type !== "webapppassword") return;
  if (event.origin !== new URL(server).origin) return; // always verify
  auth = data;
  authReconnect = false;
});

function authHeader() {
  return "Basic " + btoa(auth.loginName + ":" + auth.token);
}

let authReconnect = false; // reconnect at most once per session

async function davFetch(url, options) {
  let opts = Object.assign({}, options, {
    headers: Object.assign({ Authorization: authHeader() }, options.headers)
  });
  let res = await fetch(url, opts);
  if (res.status === 401 && !authReconnect) {
    authReconnect = true;
    connect();                       // user re-authenticates in the tab
    await new Promise(r => setTimeout(r, 120000)); // wait for the message
    if (!auth) throw new Error("Authentication not completed");
    opts = Object.assign({}, options, {
      headers: Object.assign({ Authorization: authHeader() }, options.headers)
    });
    res = await fetch(url, opts);
  }
  return res;
}

// List a folder
async function listFolder(relPath) {
  const base = auth.webdavUrl.replace(/\/+$/, "");
  const url = relPath ? base + "/" + relPath.split("/").map(encodeURIComponent).join("/") : base;
  const res = await davFetch(url, { method: "PROPFIND", headers: { Depth: "1" } });
  if (!res.ok) throw new Error("HTTP " + res.status);
  return await res.text(); // parse XML: href + resourcetype/collection
}

// Upload a file (creates missing parent folders)
async function uploadFile(relPath, content, contentType) {
  const base = auth.webdavUrl.replace(/\/+$/, "");
  const url = base + "/" + relPath.split("/").map(encodeURIComponent).join("/");
  const parts = relPath.split("/").slice(0, -1);
  let built = "";
  for (const seg of parts) {
    built = built ? built + "/" + seg : seg;
    await davFetch(base + "/" + built.split("/").map(encodeURIComponent).join("/"), { method: "MKCOL" });
  }
  const res = await davFetch(url, {
    method: "PUT",
    headers: { "Content-Type": contentType },
    body: content
  });
  if (!res.ok) throw new Error("HTTP " + res.status);
}
```

---

## 6. Security Notes

- **Origin verification:** never accept the postMessage payload without checking `event.origin` against the Nextcloud server origin.
- **Token lifetime:** temporary app passwords expire; handle `401` by re-running the connect flow (at most once per session to avoid loops).
- **Client-side only:** store the token in browser storage; never send it to your own backend or commit it.
- **Allowlist scoping:** keep the allowed origins list as narrow as possible; wildcards cover every subdomain of a domain, so an origin on a shared or untrusted subdomain would also be allowed.
- **Popups:** the connect window must open from a user gesture; be prepared for popup blockers (provide a fallback message and a manual "Connect" action).
