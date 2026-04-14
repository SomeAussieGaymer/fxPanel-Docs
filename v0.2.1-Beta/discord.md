# Discord Integration

Status embeds, slash commands, and admin notifications for your Discord server.

---

## Status Embed

fxPanel can post a persistent, auto-updating embed in a Discord channel that shows your server's live status. The embed updates every minute with current player count, uptime, and server information.

### Setup

1. Configure your Discord Bot token in fxPanel Settings → Discord Bot.
2. Use the `/status add` slash command in the channel where you want the embed to appear.
3. The embed will be created and begin updating automatically.

> **Tip:** You can fully customize the embed's appearance using the standalone JSON embed editor in fxPanel Settings. This supports custom fields, colors, footer text, and up to 5 buttons with emoji.

---

## Placeholders

Use these dynamic placeholders in your embed JSON. They are replaced with live values on each update.

| Placeholder | Description |
|-------------|-------------|
| `{{serverCfxId}}` | Server CFX ID |
| `{{serverJoinUrl}}` | Direct connect URL |
| `{{serverBrowserUrl}}` | Server browser URL |
| `{{serverClients}}` | Current player count |
| `{{serverMaxClients}}` | Maximum player slots |
| `{{serverName}}` | Server name |
| `{{statusColor}}` | Status color code (green/yellow/red) |
| `{{statusString}}` | Status text (Online, Offline, etc.) |
| `{{uptime}}` | Server uptime formatted |
| `{{nextScheduledRestart}}` | Time until next scheduled restart |

---

## Slash Commands

The fxPanel Discord bot registers these slash commands:

**`/status add`**
Create a persistent, auto-updated status embed in the current channel.

**`/status remove`**
Remove the configured persistent status embed.

**`/whitelist member <member>`**
Add a Discord member to the whitelist pre-approvals.

**`/whitelist request <id>`**
Approve a whitelist request by its ID (e.g. R1234).

**`/info <self | member | id>`**
Search for a player in the fxPanel database and display their information. Use `self` to look up yourself, `member` for a Discord user, or `id` for an identifier (e.g. `fivem:271816`).

**`/admininfo <self | member | id>`**
Admin-only player search that shows identifiers, notes, and action history. Same subcommands as `/info`.
