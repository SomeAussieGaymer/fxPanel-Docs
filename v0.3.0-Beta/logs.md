# Logging

Admin logs, FXServer console logs, in-game server logs, and optional Discord log routing.

---

## Log Types

fxPanel maintains several log channels accessible from the web panel.

### Admin Logs

All administrative actions are logged automatically, including:

- Player bans, warns, kicks, and revocations
- Server starts, stops, and restarts
- Settings changes
- Admin account modifications
- Resource start/stop commands
- Announcements and direct messages

Admin logs are viewable from the **System Logs** page by admins with `txadmin.log.view` permission. Each entry includes the admin name, action type, timestamp, and any relevant metadata (target player, reason, duration, etc.).

### FXServer Console Log

The live FXServer console output is available in the **Console** page. Admins with `console.view` can read the output, and those with `console.write` can execute commands. The console log is kept in a circular buffer — older messages are discarded when the buffer fills.

### Server Logs

In-game activity captured by the `monitor` resource logger. These appear in the **Server Log** page and can be filtered by event type and player context.

### Discord Log Routes

The standalone Discord bot can mirror selected fxPanel logs into Discord channels.

Configure this in **Settings -> Discord Bot -> Edit Logging**.

That editor also includes:

- one shared **Log Guild Override** that applies to all Discord log routes
- a dedicated **Warnings Channel** entry in the left-side list for restart and warning-style announcement messages

Each route can:

- be enabled independently
- target its own Discord channel ID

Built-in routes currently include:

- panel action logs
- panel command logs
- admin login logs
- config change logs
- monitor logs
- scheduler logs
- other system logs
- in-game admin command logs

The **Admin Command Logs** route supports an advanced filter so you can allow or block individual in-game commands such as noclip, teleport, heal, spectate, vehicle tools, and troll actions.

Discord embeds for in-game admin commands include the admin username, command, permission, location, and time.

---

## Configuration

Logging behavior can be configured in fxPanel Settings:

- **Log retention** — How long admin action logs are kept before being pruned
- **Console buffer size** — How many lines of FXServer console output to keep in memory
- **Server log storage** — Rotating file stream options, retention days, and event types to exclude from server logging
- **Discord log routing** — The warnings channel, one shared log guild override, which log streams post to Discord, their channel IDs, and advanced in-game command filters

---

## Custom Server Logs

The Server Log page is fed by the `monitor` resource logger. It records built-in events such as player joins/leaves, chat messages, explosions, menu actions, death notices, and selected fxPanel system activity.

There is no current `txAdmin:events:serverLog` public API in the source. For compatibility with older integrations, the resource still accepts these server-side events from the server console/source `0`:

- `txaLogger:CommandExecuted`
- `txaLogger:DebugMessage`

Those compatibility events are marked deprecated in the source and log a warning the first time they are used.

Server log entries include a timestamp, event type, source player or `tx`, and event-specific data. Filtering is based on the event type and player context exposed by the Server Log UI.
