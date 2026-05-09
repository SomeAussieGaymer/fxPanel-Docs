# Logging

Admin logs, FXServer console logs, and custom server log events.

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

Custom server log events sent by your resources via the logging API. These appear in the **Server Log** page and can be filtered by source and type.

---

## Configuration

Logging behavior can be configured in fxPanel Settings:

- **Log retention** — How long admin action logs are kept before being pruned
- **Console buffer size** — How many lines of FXServer console output to keep in memory
- **Server log storage** — Where custom server log entries are stored and for how long

---

## Custom Server Logs

Resources can send custom log entries to fxPanel's Server Log system using the `txAdmin:events:serverLog` event pattern. This lets you capture in-game activity in a centralized, searchable log.

### How to Enable

1. Emit a server event with your log data from any server-side resource
2. The entry appears in the Server Log page in the web panel
3. Entries can be filtered by the source resource name

### Example

```lua
-- Send a custom server log entry
TriggerEvent('txAdmin:events:serverLog', {
    source = 'my-resource',
    type = 'info',
    message = 'Player purchased a vehicle: Adder',
    data = {
        player = GetPlayerName(source),
        vehicle = 'adder',
        price = 1500000
    }
})
```

Log entries include a timestamp, source resource name, log level (info/warn/error), message text, and optional structured data.
