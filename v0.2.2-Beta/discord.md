# Discord Integration

Status embeds, slash commands, admin notifications, and OAuth login for your Discord server.

---

## Discord OAuth Login

fxPanel supports signing in with Discord — if an admin account has a matching Discord ID, they can log in directly from the login page using their Discord account instead of a username/password.

### Setup

1. Go to the [Discord Developer Portal](https://discord.com/developers/applications) and select your application (you can reuse the same one as your bot).
2. Navigate to **OAuth2** in the sidebar.
3. Copy the **Client ID** and **Client Secret**.
4. Under **Redirects**, add your panel URL followed by `/login/discord/callback`:
   ```
   https://your-panel-url.com/login/discord/callback
   ```
5. In fxPanel, go to **Settings → Discord Bot** and fill in the **OAuth Client ID** and **OAuth Client Secret** fields.
6. Save. The "Login with Discord" button will now appear on the login page.

### How It Works

- When a user clicks "Login with Discord", they are redirected to Discord's authorization page requesting the `identify` scope (read-only access to their Discord user ID and username — no server or message access).
- After authorizing, Discord redirects back to your panel with a temporary code.
- fxPanel exchanges the code for the user's Discord ID and matches it against admin accounts that have a Discord ID configured in their profile.
- If a match is found, a 24-hour session is created — identical to the Cfx.re login flow.

### Requirements

- The admin must have a **Discord ID** set on their account (configured in Admin Manager or self-service identifier settings).
- Both **OAuth Client ID** and **OAuth Client Secret** must be set in Settings → Discord Bot.
- The redirect URL must exactly match what's configured in the Discord Developer Portal.

> **Note:** The Discord bot does not need to be enabled for OAuth login to work. The OAuth Client ID/Secret are separate from the bot token.

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
