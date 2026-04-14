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
- The menu requires the `menu.access` permission at minimum
- Check that the resource is started: type `restart monitor` in the server console
- Verify the NUI hasn't been blocked by another resource

### Menu opens but shows a blank screen

- Another resource may be interfering with the NUI layer — check for `SetNuiFocus` conflicts
- Try restarting the monitor resource: `restart monitor`
- Check the F8 console for JavaScript errors

### Keybinds not responding

- Default keybind is `F1` (configurable via ConVars)
- Some resources override keybinds — check for conflicts
- Verify the menu is enabled for your admin role in the Permissions page

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

### Slash commands not appearing

- Slash commands may take up to 1 hour to propagate after bot setup
- Ensure the bot has the `applications.commands` scope
- Try kicking and re-inviting the bot

---

## Addons

### Addon isn't discovered

- Verify the directory name matches the `id` field in `addon.json`
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
- SDK import must be `from 'addon-sdk'` (not a relative path)
- Missing `"fxpanel": { "minVersion": "..." }` in the manifest

### API routes return 503

- The addon process is not running (check state in the Addons page)
- The addon crashed — check console logs for the error

### Public routes return "Not found"

- Ensure `"publicRoutes": true` is set in your `addon.json` manifest
- Verify your route paths use simple `:param` segments — regex patterns like `:path(.*)` are not supported
- Check that the number of path segments in the request matches the registered route pattern

### Dedicated public port not starting

- Check if another process is already using the configured port
- Verify the port number is valid (1024–65535)
- Check the console for `[addon:<id>]` bind errors
- If no port is configured, public routes are available via the `/site/<addonId>/` fallback on the main port

### Storage operations fail

- Ensure `storage` is listed in `permissions.required` and was approved
- Storage operations timeout after 5 seconds
- Check if the addon has exceeded the 10 MB storage limit

### Panel component doesn't render

- Verify your entry file exports a function matching the `component` name in the manifest
- Check the browser console for import or render errors
- Ensure React is not bundled in your panel entry (use externals)

### WebSocket push not working

- Ensure `ws.push` is in your granted permissions
- Panel clients must be subscribed to the addon's room (automatic for addon components)

### Hot-reload not triggering

- Ensure file changes are within the addon's directory
- Check that the addon is in the `running` state
- Panel/NUI bundles are not hot-reloaded — refresh the browser manually after rebuilding