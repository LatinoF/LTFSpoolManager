# LTF Spool Manager - Changelog

## v1.1.2 - 2026-08-02

**Description:** Cloud Backup authentication rewritten to the WebAppPassword flow: the app now connects to Nextcloud via a login popup and uses a temporary app password (WebAppPassword Nextcloud app, which also enables the required CORS headers), instead of stored username/password Basic auth, which the browser cannot use cross-origin. Automatic reconnection on token expiry (once per session), legacy stored credentials dropped on load, and the connect/disconnect UI replaces the old credential fields.

**Type:** patch

**Notes:**
- Requires the official WebAppPassword app (v26.6.0) installed on Nextcloud with allowed origins configured (e.g. `https://*.latinof.com`).
- Temp app passwords expire; the app reconnects automatically once per session on 401, otherwise prompts to reconnect.
- Credentials (temp token + login name) remain client-side only (localStorage).

## v1.1.1 - 2026-08-02

**Description:** Cloud backup UI overhaul: uniform 46px control height across all modals (inputs, selects, buttons, chips), square icon-only Browse/Create/Back buttons, Test/Backup/Restore on a single row, "Restore from cloud..." shortened to "Restore", "App password" placeholder renamed to "Password", and the Cloud Backup settings button moved to the top of the settings submenu.

**Type:** patch

**Notes:**
- No behavior changes: auth, daily backup, restore, and folder picker logic are untouched.

## v1.1.0 - 2026-08-02

**Description:** Added automatic daily cloud backup to Nextcloud via WebDAV (with folder browser picker, manual backup, and restore from cloud). Full English localization of code, comments, identifiers, and UI strings, with automatic migration of legacy localStorage data (old Italian keys converted to the new English schema). Removed broken background image reference. Added project .gitignore.

**Type:** minor

**Notes:**
- Cloud credentials are stored only in the client browser (localStorage), never on the server or repository.
- Backup files are named `YYYY.MM.DD_LTF_SpoolManager_DB.json` and accumulate in the chosen Nextcloud folder.
- Daily backup runs automatically on page load when the app is opened on a new day; failed uploads are retried on the next page load.
- Nextcloud requires `'cors.allowed-domains' => ['https://<your-domain>']` in `config.php` and an app password instead of the account password.
- Existing localStorage data is migrated automatically on first load after the update; no data loss.
- Restore from cloud is manual only (Settings > Cloud Backup > Restore from cloud).
