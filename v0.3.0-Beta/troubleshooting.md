# Troubleshooting

Fixes for common installation, panel, in-game menu, Discord, and addon problems. For literal error strings from the multi-instance host, HTTP API, Admin Manager, and Discord OAuth, see [Error messages](/docs/v0.3.0-Beta/troubleshooting#error-messages).

---

## Installation

### fxPanel doesn't start / "monitor" resource not found

- Ensure you placed the fxPanel `monitor` folder inside `citizen/system_resources/`, not in the server data folder
- Verify you fully deleted the old `monitor` folder — leftover files can cause conflicts
- Check that the directory structure is `citizen/system_resources/monitor/` with `fxmanifest.lua` at the root

### Port already in use

- The default port is `40120`. If another process is using it, set a different port via the `TXHOST_TXA_PORT` environment variable (see [Networking](/docs/v0.3.0-Beta/configuration#networking))
- On Linux, use `lsof -i :40120` or `ss -tlnp | grep 40120` to find the conflicting process
- On Windows, use `netstat -ano | findstr :40120` to identify the process

### "Cannot find module" errors on startup

- Ensure you're using a compatible FXServer artifact version — use the [Artifact Updater](/docs/v0.3.0-Beta/artifact-updater) if you need a newer runtime build
- Re-download and re-extract fxPanel — the archive may have been corrupted
- Verify Node.js modules weren't accidentally deleted from the monitor directory

---

## Web Panel & Login

### Can't access the panel at localhost:40120

- Verify FXServer is running and fxPanel initialized (check the server console for the "fxPanel is ready" message)
- Try `http://127.0.0.1:40120` instead of `localhost`
- Check your firewall isn't blocking port 40120
- If hosting remotely, use your server's public IP instead of localhost

### Login fails / "Invalid credentials"

- The master account is created on first launch — if you skipped it, delete the `txData` profile folder and restart
- Passwords are case-sensitive
- Clear browser cookies and try again
- Check the server console for authentication error messages

### CSRF token errors

- Clear all cookies for the panel domain and log in again
- This can happen if you have multiple tabs open and one session expired
- If using a reverse proxy, ensure it's forwarding the `X-TxAdmin-CsrfToken` header

---

## In-Game Menu

### /tx command doesn't work

- Ensure the player has admin permissions configured in the panel
- The menu requires the relevant in-game permissions for each feature, such as `menu.viewids`, `menu.vehicle.spawn`, or `players.noclip`
- Check that the resource is started: type `restart monitor` in the server console
- Verify the NUI hasn't been blocked by another resource

### Menu opens but shows a blank screen

- Another resource may be interfering with the NUI layer — check for `SetNuiFocus` conflicts
- Try restarting the monitor resource: `restart monitor`
- Check the F8 console for JavaScript errors

### Keybinds not responding

- fxPanel registers keybind commands, but the default key is empty; assign a key in Game Settings -> Key Bindings -> FiveM
- Some resources override keybinds — check for conflicts
- Verify the menu is enabled and that your admin has the feature-specific permission for the keybind action

---

## Discord Integration

See [Discord Integration](/docs/v0.3.0-Beta/discord) for setup (standalone bot, OAuth login, logging, and slash commands). OAuth login issues are listed under [OAuth errors and API responses](/docs/v0.3.0-Beta/discord#oauth-errors-and-api-responses) and in [Error messages](/docs/v0.3.0-Beta/troubleshooting#error-messages) below.

### Bot doesn't come online

- Double-check the bot token in Settings → Discord (see [Standalone Bot Runtime](/docs/v0.3.0-Beta/discord#standalone-bot-runtime))
- Ensure the bot has been invited to your Discord server with the correct permissions
- The bot requires the `Send Messages` and `Embed Links` permissions at minimum
- Check the server console for Discord connection errors

### Status embed not updating

- Verify the target channel ID is correct
- The bot needs `Send Messages` and `Embed Links` permissions in the specific channel
- Status embeds update on an interval — wait at least 60 seconds
- Check for rate-limiting messages in the server console

### Discord logs not posting

- Verify the correct route is enabled in **Settings -> Discord Bot -> Edit Logging** (see [Discord Logging](/docs/v0.3.0-Beta/discord#discord-logging))
- If the issue is with restart or warning-style announcements, open the **Warnings Channel** entry in that list and verify its channel ID
- Double-check the route's channel ID and the shared log guild override if you are sending logs to a different Discord server
- If you enabled advanced filtering for **Admin Command Logs**, make sure the specific command is still checked
- The bot needs `Send Messages` and `Embed Links` permissions in the target channel
- Check the server console for bridge or Discord delivery errors

### Slash commands not appearing

- Slash commands may take up to 1 hour to propagate after bot setup
- Ensure the bot has the `applications.commands` scope
- Try kicking and re-inviting the bot
- For addon-provided commands, ensure the addon is in the `running` state and that `discordBot.commands` points to an addon-relative path with no absolute segments or `..`

---

## Addons

### Addon isn't discovered

- Verify the manifest `id` is unique and matches the required lowercase ID format
- Check the console for Zod validation errors on the manifest
- Ensure `addon.json` exists in the addon root (not in a subdirectory)
- The `id` must be lowercase with only `a-z`, `0-9`, and hyphens (3-64 chars)

### Addon stays in "discovered" state

- It needs to be approved by an admin with `all_permissions`
- After approval, the addon starts on the next boot or hot-reload cycle

### Addon fails to start

- Check the console for `[addon:<id>]` error messages
- Missing `"type": "module"` in `package.json`
- Syntax error in the server entry file
- `addon.ready()` never called (10-second timeout)
- Server entries must import `from 'addon-sdk'`; addon-owned Discord command/event files should import `from 'addon-sdk/discord'` (not a relative path)
- Invalid `discordBot.commands` or `discordBot.events` path in `addon.json` (absolute paths and `..` traversal are rejected)
- Missing `"fxpanel": { "minVersion": "..." }` in the manifest

### API routes return 503

- The addon runtime is not running (check state in the Addons page)
- The addon crashed — check console logs for the error

### Public routes return "Not found"

- Ensure `"publicRoutes": true` is set in your `addon.json` manifest
- Verify your route paths use simple `:param` segments — regex patterns like `:path(.*)` are not supported
- Check that the number of path segments in the request matches the registered route pattern

### Dedicated public port not starting

- Check if another process is already using the configured port
- Verify the configured port number is valid (1-65535). In practice, ports below 1024 may require elevated privileges on some hosts.
- If fxPanel logs a TLS credential error, set `TXHOST_ADDON_PUBLIC_TLS_KEY` + `TXHOST_ADDON_PUBLIC_TLS_CERT` or `TXHOST_ADDON_PUBLIC_TLS_KEY_FILE` + `TXHOST_ADDON_PUBLIC_TLS_CERT_FILE`
- Check the console for `[addon:<id>]` bind errors
- If no port is configured, fxPanel logs that public routes are enabled without a public server port. The main panel origin intentionally does not serve public addon routes.

### Storage operations fail

- Ensure `storage` is listed in `permissions.required` and was approved
- Storage operations timeout after 5 seconds
- Check if the addon has exceeded the configured storage limit (10 MB by default)

### Panel component doesn't render

- Verify your entry file exports a function matching the `component` name in the manifest
- Check the browser console for import or render errors
- Ensure React is not bundled in your panel entry (use externals)

### Theme addon doesn't apply on the login page

- Verify the addon is in the `running` state (check the Addons page)
- Ensure `static/theme.json` exists in the addon's directory with `"enabled": true`
- The `panel` object in `theme.json` must contain CSS custom property names starting with `--`
- CSS selectors must be gated behind a data attribute (e.g. `html[data-addon-themer-enabled='true']`) — the server sets this attribute when a valid `theme.json` is found
- Refresh the page after making changes to `theme.json` — values are read at HTML generation time

### Theme addon logo doesn't appear on the login page

- Ensure `branding.panelLogo` in `theme.json` points to a valid file in the addon's `static/` directory
- Supported formats: PNG, JPG, GIF, SVG, WebP, ICO
- The logo is base64-encoded and embedded in the HTML. Files larger than 100,000 bytes are skipped.
- Check that the filename in `theme.json` matches the actual file exactly (case-sensitive on Linux)

### WebSocket push not working

- Ensure `ws.push` is in your granted permissions
- Panel clients must be subscribed to the addon's room (automatic for addon components)

### Hot-reload not triggering

- Ensure file changes are within the addon's directory
- Check that the addon is in the `running` state
- Panel/NUI bundles are not hot-reloaded — refresh the browser manually after rebuilding

---

## Error messages

Literal strings returned by fxPanel, the standalone multi-instance host (`fxp-host`), HTTP APIs, and the panel UI. Use this section when logs or toasts match a message exactly. Related guides: [Configuration](/docs/v0.3.0-Beta/configuration), [Admin Manager](/docs/v0.3.0-Beta/admin-manager), [Artifact Updater](/docs/v0.3.0-Beta/artifact-updater), [Discord Integration](/docs/v0.3.0-Beta/discord).

### Host (FXPHOST)

Console stderr / startup failures for the standalone host process (`packages/host`).

| Message / pattern | Typical cause | What to do |
| ----------------- | ------------- | ---------- |
| `EADDRINUSE binding instance "<id>" UI on <port>` | Another process (or second `fxp-host`) owns `fxpUiPort` | Stop the conflicting listener; adjust `fxpUiPort` / `FXPHOST_UI_PORT_BASE` in `host.instances.json`; or set `FXPHOST_UI_LISTEN_RELAX=1` for local dev |
| `[fxp-host] instance "<id>": fxpUiPort <n> busy; listening on <m> (FXPHOST_UI_LISTEN_RELAX)` | Relax mode shifted the bind to the next free port | Update `host.instances.json` if you need a fixed port; otherwise expected in dev |
| `Could not bind extra UI for instance "<id>" starting at fxpUiPort <n>` | No free port in the retry window | Free ports or raise offsets / disable conflicting instances |
| `fxpUiPort <p> conflicts with FXPHOST_HTTP_PORT` | Instance panel port equals control-plane HTTP port | Change `FXPHOST_HTTP_PORT` or the instance `fxpUiPort` |
| `Duplicate fxpUiPort <p>` | Two instances share the same UI port | Assign unique `fxpUiPort` values |
| `Gameplay port <g> equals FXPHOST_HTTP_PORT` / `equals an fxpUiPort` | Game port collides with host or panel ports | Adjust game endpoint / `FXPHOST_GAME_PORT_BASE` or instance ports |
| `Admin API auth is ON (FXPHOST_ADMIN_TOKEN)` | Informational — token auth enabled | Send `Authorization: Bearer …`, `X-Fxp-Admin-Token`, or use `POST /api/v1/session` |

Environment variables for the host are summarized in the host operator docs; panel-side `TXHOST_*` validation is [below](#txhost-environment-validation).

### Instances (host API)

JSON `error` fields from `POST /api/v1/instances` and instance lifecycle routes.

| Message / pattern | HTTP | Typical cause | What to do |
| ----------------- | ---- | ------------- | ---------- |
| `Instance id already exists: <id>` | 409 | Duplicate `id` in `host.instances.json` | Pick a new id or remove the old row |
| `Unknown instance: <id>` | 404 | Stale id or typo | Refresh instance list; fix client id |
| `Already running` | 409 | Start requested while FXServer is up | Stop first or use restart |
| `Still running after stop` | 409 | Stop did not terminate the process | Kill orphaned FXServer; check `serverCwd` / binary path |
| `Persisted instance not resolved after write` | 500 | Host failed to reload registry after create | Check disk permissions on instances file; restart host |
| `config.json not found or unreadable` | 404 | Profile path wrong for instance | Fix `serverCwd` / deploy profile |
| Zod `flatten()` object on create body | 400 | Invalid JSON body | Match OpenAPI / `createInstanceBodySchema` fields |

Port-plan violations from instance validation use the same codes as [Host (FXPHOST)](#host-fxphost) (`duplicate_fxp_ui_port`, `fxp_ui_equals_control_plane`, etc.).

### Artifact download and apply (core)

Background updater and `/fxserver/artifacts/*` routes (`core/modules/FxUpdater`).

| Message / pattern | Where | Typical cause | What to do |
| ----------------- | ----- | ------------- | ---------- |
| `Only admins with all permissions can manage artifacts.` | Download route toast | Missing `all_permissions` | Grant permission or use master account |
| `A valid HTTPS download URL is required.` | Download route | Bad URL | Use `https://runtime.fivem.net/...` only |
| `Download URL hostname is not allowed. Permitted: runtime.fivem.net` | Download route | Custom host blocked | Use official artifact URLs |
| `A version identifier is required.` | Download route | Missing `version` field | Pass tier label or `custom` |
| `No downloaded update ready to apply. Please download first.` | Apply route | Status not `extracted` | Complete download/extract first |
| `A download is already in progress.` | Updater (internal) | Overlapping download | Wait for current job |
| `An update is currently being applied.` | Updater (internal) | Apply in progress | Wait for restart |
| `Invalid artifact structure: …` / `FXServer.exe not found` | Extract phase | Wrong platform archive | Pick Windows vs Linux tier; see [Artifact Updater](/docs/v0.3.0-Beta/artifact-updater#platform-behavior) |
| `tar exited with code <n>` | Linux extract | Corrupt `.tar.xz` | Re-download; check disk space |
| `Archive entry has unsafe name` / `escapes staging directory` | ZIP extract | Zip-slip attempt | Do not use untrusted archives |
| `Failed to stop game server: …` | Apply | FXServer did not exit | Stop server manually; retry apply |
| `fxserver_update_failure.txt` beside artifact root | Persisted failure | Prior download/apply failed | Read file; fix cause; delete marker and retry |

Panel UI strings for `/system/artifacts` are in [Artifact updater UI](#artifact-updater-ui).

### HTTP / SQL (host API)

`GET /host/status` (embedded monitor) and `/api/v1/*` database routes.

| Message / pattern | Where | Typical cause | What to do |
| ----------------- | ----- | ------------- | ---------- |
| `token not configured` | `GET /host/status` | `TXHOST_API_TOKEN` unset | Set token or `disabled` — see [Configuration](/docs/v0.3.0-Beta/configuration#environment-variables) |
| `token missing` / `invalid token` / `token conflict` | `GET /host/status` | Wrong/missing `X-TxAdmin-EnvToken` or query `envtoken` | Match env token; use header **or** query, not both |
| `Unauthorized` + hint about `FXPHOST_ADMIN_TOKEN` / bridge | Host API 401 | No admin or bridge token | Set `FXPHOST_ADMIN_TOKEN`; use Bearer, `X-Fxp-Admin-Token`, or session cookie |
| `FXPHOST_ADMIN_TOKEN is not set on this host` | `POST /api/v1/session` | Session login disabled | Set admin token on host |
| `Invalid token` | `POST /api/v1/session` | Wrong body token | Copy exact token from host env |
| `sqlite_required` + `Set FXPHOST_DB_MODE=sqlite` | Profile mirror routes 503 | SQLite mode off | Set `FXPHOST_DB_MODE=sqlite` and optional mirror flags |
| `MariaDB is not fully configured (need host, database, user via FXPHOST_DB_*)` | MariaDB query route | Missing DB env | Set `FXPHOST_DB_HOST`, `FXPHOST_DB_NAME`, `FXPHOST_DB_USER`, etc. |
| `Invalid SQL` | MariaDB query route | Parser rejected statement | Fix SQL; avoid multi-statement abuse |
| `Enable FXPHOST_DB_MODE=sqlite for audit persistence.` | `GET /api/v1/audit/recent` | Audit DB disabled | Informational when mode is `none` |

### Admin Manager

Panel toasts from `POST /adminManager/:action` (requires `manage.admins` unless noted).

| Message / pattern | Typical cause | What to do |
| ----------------- | ------------- | ---------- |
| `You don't have permission to execute this action.` | Missing `manage.admins` | Grant permission — see [Permissions](/docs/v0.3.0-Beta/permissions) |
| `Invalid Request` / `Invalid Request - missing parameters` | Malformed API body | Check required fields (`name`, `permissions`, …) |
| `Invalid username, it must follow the rule:` | Username regex failed | 3–20 chars; see [Account Constraints](/docs/v0.3.0-Beta/admin-manager#account-constraints) |
| `Invalid CitizenFX ID1` / `ID2` / `ID3` / `(ERR1–3) Invalid CitizenFX ID` | Cfx.re lookup failed | Use `fivem:<id>` or resolvable forum username |
| `Failed to verify CitizenFX ID. Please try again or check the ID.` | Network/API error | Retry; verify id |
| `Invalid Discord ID` | Snowflake regex failed | Use numeric Discord user id |
| `You cannot give permissions you do not have:` | Privilege escalation blocked | Grant only permissions you hold |
| `(ERR0) You cannot edit yourself.` | Self-edit blocked | Use Account settings or another admin |
| `You can't delete yourself.` / `You cannot delete an admin master.` | Safety rule | See [Safety Rules](/docs/v0.3.0-Beta/admin-manager#safety-rules) |
| `Admin not found.` | Wrong username | Refresh admin list |
| `You cannot reset your own password here.` | Use change-password flow | Open profile password page |
| `Unknown action.` | Bad `:action` segment | Use `add`, `edit`, `delete`, `resetPassword` |

More context: [Common errors (Admin Manager)](/docs/v0.3.0-Beta/admin-manager#common-errors).

### Discord OAuth

Login flow (`/auth/discord/*`) — setup in [Discord OAuth Login](/docs/v0.3.0-Beta/discord#discord-oauth-login).

| Message / `errorCode` | Typical cause | What to do |
| --------------------- | ------------- | ---------- |
| `Discord OAuth is not configured.` | Missing client id/secret | Set OAuth fields under Settings → Discord Bot |
| `no_admins_setup` (redirect API) | No admins in vault yet | Complete master setup first |
| `invalid_session` | Session lost between redirect and callback | Retry login; avoid clearing cookies mid-flow |
| `invalid_state` | CSRF state mismatch | Retry; one tab only |
| `Discord token exchange failed` + status | Bad code, secret, or redirect URI | Match Developer Portal redirect to `/login/discord/callback` exactly |
| `Invalid access_token in response` | Discord returned unexpected JSON | Check app OAuth settings |
| `Failed to fetch Discord user info` | User endpoint error | Retry; check Discord API status |
| `not_admin` + identifier context | Discord user not linked to an admin | Add `discord:<id>` on admin account |
| `Failed to login` | Vault/session error | Check server logs |

Full callback matrix: [OAuth errors and API responses](/docs/v0.3.0-Beta/discord#oauth-errors-and-api-responses).

### fxPanel v1 monitor sync

`POST /api/v1/host/fxpanel-v1/sync-monitor` (desktop orchestrator).

| Message / pattern | HTTP | Typical cause | What to do |
| ----------------- | ---- | ------------- | ---------- |
| `Missing v1 path: PATCH … panelV1SourcePath or pass v1SourcePath` | 400 | No v1 checkout path | Set orchestrator settings or body `v1SourcePath` |
| `Missing artifactRoot: set FXPHOST_ARTIFACT_ROOT …` | 400 | No artifact tree | Set env or `artifactRoot` in body |
| `v1 path does not exist: <path>` | 400 | Wrong repo path | Point at fxPanel v1 root containing `monitor/` |
| `artifactRoot does not exist: <path>` | 400 | Artifact dir missing | Fix `FXPHOST_ARTIFACT_ROOT` |
| `Expected a fxPanel v1 repo root (contains monitor/)` | 400 | Path is not v1 layout | Use repo root or `monitor` folder |
| `{ ok: false, error: "<msg>" }` | 400 | Copy/sync failure | Check permissions; disk space |

### TXHOST environment validation

Fatal startup errors from `getHostVars()` (quoted values or Zod failures). Summaries also appear on [Configuration → Environment validation errors](/docs/v0.3.0-Beta/configuration#environment-validation-errors).

| Message / pattern | Typical cause | What to do |
| ----------------- | ------------- | ---------- |
| `Invalid value for a TXHOST environment variable.` + quoted value hint | Value wrapped in `"` or `'` | Remove quotes in service manager / panel |
| Zod message on `TXHOST_API_TOKEN` | Token length/format | 16–48 chars; alphanumeric, `_`, `-`; or `disabled` |
| `DATA_PATH must be an absolute path` | Relative `TXHOST_DATA_PATH` | Use full path |
| `TXA_PORT cannot be 30120` | Reserved port | Pick another panel port |
| `FXS_PORT cannot be between 40120 and 40150` | Conflicts with panel range | Change game port |
| `Invalid IPv4 address` on `TXHOST_INTERFACE` | Bad bind address | Use dotted-quad IPv4 |
| Provider name regex errors | `TXHOST_PROVIDER_NAME` format | 2–16 chars; see [Provider Customization](/docs/v0.3.0-Beta/configuration#provider-customization) |
| `The account needs to be in the username:fivemId … format` | Bad `TXHOST_DEFAULT_ACCOUNT` | `user:fivemId` or `user:fivemId:bcrypt` |
| `The key needs to be in the cfxk_… format` | Bad `TXHOST_DEFAULT_CFXKEY` | Valid Cfx license key |
| `Public addon server requires TLS credentials. Set TXHOST_ADDON_PUBLIC_TLS_*` | Public addon port without TLS | Set key/cert env vars or disable public routes |

### Artifact updater UI

Strings shown on `/system/artifacts` (`FxUpdaterPage`).

| Message / pattern | Typical cause | What to do |
| ----------------- | ------------- | ---------- |
| `Failed to load artifact data` | `GET /fxserver/artifacts` failed | Check network, permissions, server logs |
| `Please enter a URL` | Empty custom URL field | Paste artifact URL |
| `Please enter a valid https URL` | Non-HTTPS or parse error | Use `https://runtime.fivem.net/...` |
| `Please enter a valid https URL from an allowed domain (runtime.fivem.net)` | Host not allowlisted | Official runtime only |
| `Downloading FXServer build <version>...` | Success toast (download started) | Wait for status poll |
| `Applying update... The server will restart shortly.` | Apply accepted | Expect process exit/restart |
| Updater status `phase: error` + `message` | Background failure | Match message in [Artifact download and apply](#artifact-download-and-apply-core); check `fxserver_update_failure.txt` |

Operator workflow: [Artifact Updater](/docs/v0.3.0-Beta/artifact-updater#operator-checklist).
