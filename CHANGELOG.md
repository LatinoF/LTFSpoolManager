# LTF Spool Manager - Changelog

## v1.1.7 - 2026-08-02 (patch)
- Change: the cloud backup status indicator in the header is now a bare icon — no background, border or box
- Change: modal buttons got hover effects (green buttons brighten with a soft glow; outline/icon buttons get the accent border and a faint blue tint), eased in/out with a 0.2s transition, without any movement

## v1.1.6 - 2026-08-02 (patch)
- Fix: Add Print Job now opens with a clean form every time (weight and description cleared, error styling and placeholder reset) instead of showing the previous error state and values
- Fix: Edit Print Job clears leftover error styling on the weight field when opened

## v1.1.5 - 2026-08-02 (patch)
- Add Print Job / Edit Print Job: invalid weight (empty, zero, negative, non-numeric) now shows a red border on the field, focuses it, clears the value and writes "Enter a weight greater than 0" in the placeholder; typing restores the normal placeholder

## v1.1.4 - 2026-08-02 (patch)
- Fix: deleting a spool from its detail view no longer reopens a stale or non-existent spool (history state replaced before returning to the dashboard)
- Fix: detail view guards against stale indexes instead of crashing
- Fix: editing a print job no longer accepts zero or empty weights
- Fix: folder/file names from WebDAV listings and user-entered text are HTML-escaped when rendered via innerHTML
- Fix: nested backup folder paths (e.g. a/b) are created segment by segment
- Change: Nunito font loaded from Google Fonts and applied, replacing the never-loaded Inter reference
- Chore: removed unused execShowDashboard parameter and unused element IDs (confirm-icon, picker-back-btn)

## v1.1.3 - 2026-08-02 (patch)
- Fix: folder selection in the Cloud Backup folder picker returned HTTP 404 (WebDAV listing paths were built relative to the wrong prefix, producing a double-username path); the restore list had the same flaw
- Change: the DAV base URL now comes from the server-provided webdavUrl in the WebAppPassword login message
- Change: the folder picker falls back to the root with a notice when a typed folder cannot be opened
- Change: the Nextcloud login now opens in a new tab instead of a popup window

## v1.1.2 - 2026-08-02 (patch)
- Change: Cloud Backup authentication rewritten to the WebAppPassword flow — a login popup provides a temporary app password (connect/disconnect buttons replace the username/password fields)
- Change: legacy stored credentials are dropped automatically on load
- Change: on token expiry (HTTP 401) the app reconnects automatically once per session, otherwise prompts to reconnect
- Dependency: requires the WebAppPassword app (v26.6.0) installed on Nextcloud with allowed origins configured (e.g. `https://*.latinof.com`); credentials stay client-side only

## v1.1.1 - 2026-08-02 (patch)
- Change: uniform 46px control height across all modals (inputs, selects, buttons, chips)
- Change: Browse, Create and Back are square icon-only buttons
- Change: Test, Backup and Restore moved to a single row; "Restore from cloud..." shortened to "Restore"; "Backup now" shortened to "Backup"
- Change: "App password" placeholder renamed to "Password"
- Change: the Cloud Backup settings button moved to the top of the settings submenu

## v1.1.0 - 2026-08-02 (minor)
- Added: automatic daily cloud backup to Nextcloud via WebDAV — folder browser picker, manual backup, restore from cloud, daily dated backups (YYYY.MM.DD_LTF_SpoolManager_DB.json), backup status indicator
- Change: full English localization — comments, identifiers and UI strings (e.g. bobine→spools, pesoAttuale→currentWeight); legacy localStorage data is migrated automatically to the new schema
- Chore: removed the broken background image reference
- Chore: added Project/.gitignore
- Chore: README updated (cloud backup feature)
