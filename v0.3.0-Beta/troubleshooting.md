# Troubleshooting

Fixes for common installation, panel, in-game menu, Discord, and addon problems.

---

## Installation

### fxPanel doesn't start / "monitor" resource not found

- Ensure you placed the fxPanel `monitor` folder inside `citizen/system_resources/`, not in the server data folder
- Verify you fully deleted the old `monitor` folder — leftover files can cause conflicts
- Check that the directory structure is `citizen/system_resources/monitor/` with `fxmanifest.lua` at the root

### Port already in use

- The default port is `40120`. If another process is using it, set a different port via the `TXHOST_TXA_PORT` environment variable
- On Linux, use `lsof -i :40120` or `ss -tlnp | grep 40120` to find the conflicting process
- On Windows, use `netstat -ano | findstr :40120` to identify the process

### "Cannot find module" errors on startup

- Ensure you're using a compatible FXServer artifact version
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

### Bot doesn't come online

- Double-check the bot token in Settings → Discord
- Ensure the bot has been invited to your Discord server with the correct permissions
- The bot requires the `Send Messages` and `Embed Links` permissions at minimum
- Check the server console for Discord connection errors

### Status embed not updating

- Verify the target channel ID is correct
- The bot needs `Send Messages` and `Embed Links` permissions in the specific channel
- Status embeds update on an interval — wait at least 60 seconds
- Check for rate-limiting messages in the server console

### Discord logs not posting

- Verify the correct route is enabled in **Settings -> Discord Bot -> Edit Logging**
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
