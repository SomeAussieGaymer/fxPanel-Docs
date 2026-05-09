# Discord Integration

Persistent status embeds, live player-list embeds, moderation slash commands, report triage, configurable log routing, admin notifications, and OAuth login for your Discord server.

---

## Standalone Bot Runtime

In v0.3.X, the Discord integration runs as a standalone bot in the workspace's `bot/` folder instead of an embedded Discord client inside core. fxPanel starts and monitors that bot process automatically, then talks to it over a local WebSocket bridge bound to `127.0.0.1`.

What this means in practice:

- Slash commands, ticket thread relays, announcements, presence updates, and persistent embeds now run through the standalone bot.
- fxPanel now maintains two separate persistent embed targets: **status** and **player list**.
- Discord log routing for panel activity and in-game admin actions also runs through the same standalone bot bridge.
- The local bridge is private to the same machine and protected by a shared secret managed by fxPanel.
- The required built-in files live under `bot/bridge/`, `bot/commands/_fxpanel/`, and `bot/events/_fxpanel/`.
- You can add your own Discord.js commands or events alongside them, and running addons can also contribute Discord modules through their `discordBot` manifest section.
- Do not remove the `bridge/` or `_fxpanel/` folders because fxPanel depends on them.

---

## Discord OAuth Login

fxPanel supports signing in with Discord. If an admin account has a matching Discord ID, they can log in directly from the login page using their Discord account instead of a username/password.

### Setup

1. Go to the [Discord Developer Portal](https://discord.com/developers/applications) and select your application. You can reuse the same application as your bot.
2. Navigate to **OAuth2** in the sidebar.
3. Copy the **Client ID** and **Client Secret**.
4. Under **Redirects**, add your panel URL followed by `/login/discord/callback`:
   ```
   https://your-panel-url.com/login/discord/callback
   ```
5. In fxPanel, go to **Settings -> Discord Bot** and fill in the **OAuth Client ID** and **OAuth Client Secret** fields.
6. Save. The **Login with Discord** button will now appear on the login page.

### How It Works

- When a user clicks **Login with Discord**, they are redirected to Discord's authorization page requesting the `identify` scope.
- After authorizing, Discord redirects back to your panel with a temporary code.
- fxPanel exchanges the code for the user's Discord ID and matches it against admin accounts that have a Discord ID configured in their profile.
- If a match is found, a 24-hour session is created, identical to the Cfx.re login flow.

### Requirements

- The admin must have a **Discord ID** set on their account.
- Both **OAuth Client ID** and **OAuth Client Secret** must be set in **Settings -> Discord Bot**.
- The redirect URL must exactly match what is configured in the Discord Developer Portal.

> **Note:** The Discord bot does not need to be enabled for OAuth login to work. The OAuth Client ID/Secret are separate from the standalone bot token and bridge configuration.

---

## Persistent Embeds

fxPanel can keep persistent Discord messages updated for both overall server status and the live player list. The embed state is stored by fxPanel core, while the standalone bot is responsible for creating the Discord messages and keeping them updated.

### Status Embed

The status embed updates automatically with live server information such as player count, uptime, and any placeholders used in its JSON.

Setup:

1. Configure your Discord Bot token in **Settings -> Discord Bot**.
2. In the **Status Embed** card, use **Change Embed JSON** and **Change Config JSON** if you want to customize the output.
3. Use `/status add` in the target channel.

Use `/status remove` to clear the saved status embed from Discord and from fxPanel's stored embed state.

### Player List Embed

The player list embed is a separate persistent message with its own JSON, config JSON, and saved message state. It can render the full live player list, page summaries, and pager buttons for browsing larger servers.

Setup:

1. In **Settings -> Discord Bot**, open the **Player List Embed** editors if you want to customize the layout.
2. Use `/players add` in the target channel.

Use `/players remove` to remove the stored player-list embed.

Notes:

- The player-list embed keeps its own saved page number and stored Discord message ID.
- Pager buttons are handled by the bot runtime, so normal Discord users can move between pages without needing panel admin permissions.
- The player-list embed is especially useful when you want a dedicated message for player names instead of cramming the list into the status embed.

---

## Embed Placeholders

Use these dynamic placeholders in your status embed JSON or player-list embed JSON. They are replaced with live values on each update.

| Placeholder | Description |
|-------------|-------------|
| `{{serverCfxId}}` | Server CFX ID |
| `{{serverJoinUrl}}` | Direct connect URL |
| `{{serverBrowserUrl}}` | Server browser URL |
| `{{serverClients}}` | Current player count |
| `{{serverMaxClients}}` | Maximum player slots |
| `{{serverAvailableSlots}}` | Remaining player slots |
| `{{serverOccupancyPercent}}` | Current slot usage percentage |
| `{{serverName}}` | Server name |
| `{{statusColor}}` | Status color code (green/yellow/red) |
| `{{statusString}}` | Status text (Online, Offline, etc.) |
| `{{uptime}}` | Server uptime formatted |
| `{{nextScheduledRestart}}` | Time until next scheduled restart |
| `{{recentJoinCount}}` | Recently joined players tracked by the live tally |
| `{{recentLeaveCount}}` | Recently disconnected players tracked by the live tally |
| `{{playerList}}` | Multiline rendered player list |
| `{{playerListInline}}` | Inline rendered player list |
| `{{playerListColumns}}` | Column-friendly rendered player list |
| `{{playerListSummary}}` | Summary such as `12 players online` |
| `{{playerListPage}}` | Current player-list page number |
| `{{playerListTotalPages}}` | Total player-list pages |
| `{{playerListPageSummary}}` | Summary of the current visible page |

`{{playerListColumns}}` is particularly useful in embed fields because the player-list renderer can expand that placeholder into multiple columns when the layout supports it.

---

## Bot Presence

The standalone bot can keep its Discord activity updated from the panel.

In **Settings -> Discord Bot**, you can configure:

- the bot's online status (`online`, `idle`, `dnd`, or `invisible`)
- the activity type (`Playing`, `Watching`, `Listening`, `Competing`, or `Custom`)
- the activity text shown in Discord
- how often fxPanel refreshes that activity text

The activity text supports these live placeholders:

- `{playerCount}`
- `{maxPlayers}`
- `{serverName}`
- `{uptime}`

---

## Permission Preset Mapping

The **Role Permission Mapping** section in **Settings -> Discord Bot** lets you link Discord roles to fxPanel permission presets.

How it works:

- Each mapping can contain one or more Discord role IDs and one permission preset.
- When a linked admin signs in, fxPanel checks their current Discord guild roles and matches them against the saved mappings.
- Every matched preset is merged together and synced onto that admin account.
- Manually assigned permissions stay on the account. fxPanel only replaces the permissions that came from Discord role mappings.

This is not a hidden runtime-only overlay anymore. The synced permissions are written onto the admin record itself so they show up in Admin Manager and stay understandable to staff reviewing that account later.

Built-in Discord bot commands that use the shared permission resolver can be authorized by either:

- a linked real fxPanel admin account
- synced permission presets coming from mapped Discord roles

Persistent embed commands, whitelist commands, ticket triage commands, and addon Discord route contexts use that resolver. Moderation commands such as `/warn`, `/kick`, `/ban`, `/unban`, `/history`, and `/notes` still resolve the requester through a linked fxPanel admin account. When a mapped Discord role is used to authorize an action, fxPanel records it in audit logs with a `[Discord]` prefix.

---

## Discord Logging

The standalone bot can forward selected fxPanel logs into Discord channels.

Open **Settings -> Discord Bot -> Edit Logging** to configure this in the dedicated logging editor.

That editor includes:

- one shared **Log Guild Override** that applies to every Discord log route on the page
- a dedicated **Warnings Channel** entry in the left-side list for restart and warning-style bot announcements

Each log route can then:

- be enabled or disabled independently
- send to its own Discord channel ID

The available built-in routes currently cover:

- panel action logs
- panel command logs
- admin login logs
- config change logs
- monitor logs
- scheduler logs
- other system logs
- in-game admin command logs

The **Admin Command Logs** route also has an advanced toggle that lets you enable or disable individual in-game actions such as noclip, teleport, heal, spectate, vehicle tools, and troll actions.

When an in-game admin command log is posted to Discord, the embed includes the admin username, command, permission, location, and time.

---

## Slash Commands

The standalone fxPanel Discord bot registers these built-in slash commands from `bot/commands/_fxpanel/`.

### Persistent Embed Commands

- **`/status add`**: Create the persistent status embed in the current channel.
- **`/status remove`**: Remove the configured persistent status embed.
- **`/players add`**: Create the persistent live player-list embed in the current channel.
- **`/players remove`**: Remove the configured persistent player-list embed.

### Whitelist Commands

- **`/whitelist member <member>`**: Add a Discord member to the whitelist pre-approvals.
- **`/whitelist request <id>`**: Approve a whitelist request by ID such as `R1234`.

### Player Lookup Commands

- **`/info self`**, **`/info member <member>`**, **`/info id <identifier>`**: Search for a player and show their basic fxPanel profile.
- **`/admininfo self`**, **`/admininfo member <member>`**, **`/admininfo id <identifier>`**: Admin-only lookup that also includes identifiers, notes, and action history.

### Moderation Commands

The moderation commands target players in one of three main ways:

- **`member`**: resolve the target from a Discord member
- **`id`**: resolve the target from an explicit identifier such as `license:...`, `fivem:...`, or `discord:...`
- **`serverid`**: resolve the target from the current in-server player ID using the live player list

The current moderation command set is:

- **`/warn member|id|serverid <reason>`**: Issue a warning.
- **`/kick member|id|serverid <reason>`**: Kick a connected player.
- **`/ban member|id|serverid <duration> <reason>`**: Ban a player.
- **`/unban action <action_id> [reason]`**: Revoke a ban by action ID such as `B1234`.
- **`/unban member|id|serverid [reason]`**: Revoke the active ban for a player target.
- **`/history self|member|id|serverid [limit]`**: Show recent moderation history. `self` is also supported here.
- **`/notes view self|member|id|serverid`**: View player notes.
- **`/notes set self|member|id|serverid <note>`**: Update player notes.

### Report Triage Commands

If the in-game reports feature is enabled, the Discord bot also provides ticket triage commands:

- **`/reports summary [id]`**: Show the queue summary or one specific ticket.
- **`/reports claim [id]`**: Claim or unclaim a ticket for yourself.
- **`/reports assign <member> [id]`**: Assign a ticket to a linked fxPanel admin.
- **`/reports resolve [id]`**: Resolve a ticket.
- **`/reports reopen [id]`**: Reopen a resolved or closed ticket.

When you run these inside a report thread, the bot can infer the current ticket from the thread channel instead of requiring an explicit ID every time.

---

## Extending The Bot

The standalone runtime is a normal Discord.js project, so you can extend it without editing fxPanel core.

- For one-off local changes, add your own command files under `bot/commands/` outside of `_fxpanel/`.
- For one-off local changes, add your own event files under `bot/events/` outside of `_fxpanel/`.
- For reusable addons, declare `discordBot.commands` and/or `discordBot.events` in `addon.json` and place those files under your addon folder, for example `discord-bot/commands/`.
- Addon Discord modules are only loaded for addons that are currently in the **running** state.
- The `discordBot.commands` and `discordBot.events` values must be addon-relative paths. Absolute paths and `..` traversal are rejected during manifest validation.

### Addon Interaction Helpers

Addon-owned Discord commands can now hook more than plain slash-command execution. The standalone bot understands addon-owned:

- slash-command autocomplete handlers
- button interactions
- modal submit interactions
- string, user, role, mentionable, and channel select menus

The supported authoring path is `createAddonDiscordSdk({ addonId, bridge })` from `addon-sdk/discord`.

That SDK now exposes:

- `discord.respondWithChoices(interaction, choices)` for autocomplete responses
- `discord.interactions.button(builder, action, options)` for addon-namespaced button IDs
- `discord.interactions.modal(builder, action, options)` for addon-namespaced modal IDs
- `discord.interactions.apply(builder, kind, action, state)` when you want to attach a custom ID to a select menu or another builder that supports `setCustomId(...)`

The runtime routes those interactions back into the addon command module by matching the action name in the custom ID against the exported handler maps (`buttons`, `modals`, `stringSelectMenus`, and so on).

### Addon Rate Limiting And Diagnostics

Addons can now define a default Discord bot rate limit in `addon.json`:

```json
{
   "discordBot": {
      "commands": "discord-bot/commands",
      "rateLimit": {
         "max": 5,
         "windowMs": 15000
      }
   }
}
```

This limit is enforced per addon, per handler, and per Discord user. Addon command modules can still narrow specific handlers with their own `rateLimit` override.

When an addon command or interaction handler throws, or when it repeatedly hits its rate limit, the standalone bot now pushes that data into Discord bot diagnostics. That means **Diagnostics -> Discord Bot** will show both:

- addon load failures
- recent addon runtime issues such as command failures, modal errors, and rate-limit denials
- Addon Discord modules receive the same bridge helper as built-in modules, but the supported addon-facing wrapper is `createAddonDiscordSdk({ addonId, bridge })` from `addon-sdk/discord`.
- `createAddonDiscordSdk(...).addonRoute(...)` can build requester payloads from an interaction automatically, or you can pass explicit requester fields when you are inside a non-interaction event.
- Common addon-facing helpers are available through the same wrapper: `request(...)`, `send(...)`, `getRequesterPayload(...)`, `getConfigSnapshot()`, `resolveMemberRoles(uid)`, `resolveMemberProfile(uid)`, `refreshMemberCache()`, and `reloadCommands()`.
- `createMockDiscordBridge(...)` lets you test addon commands or events without a live bot runtime.
- If you want to validate a manifest locally before reloading the bot, `addon-sdk/discord` also exports `validateAddonDiscordManifest(...)` and `parseAddonDiscordManifest(...)`.
- The **Diagnostics -> Discord Bot** page shows addon load failures and exposes **Reload addons** and **Resync runtime** actions for the standalone bot.
- Keep the built-in `bridge/`, `_fxpanel/commands`, and `_fxpanel/events` files intact so fxPanel can keep the panel features working.

For the full addon command example, event example, mock-bridge workflow, and manifest snippets, see [Addon Development](addon-development.md).
