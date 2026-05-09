# In-Game Menu

Access admin tools directly from inside the game.

---

## Accessing the Menu

The in-game admin menu can be opened in two ways:

- **Chat Command:** Type `/tx` or `/txadmin` in the game chat. You can optionally pass a player server ID to open the menu focused on that player.
- **Keybind:** Go to Game Settings → Key Bindings → FiveM and assign a key to the **(fxPanel) Menu: Open Main Page** option.

> **Requirement**
>
> You must have an admin account with either your Discord or Cfx.re identifiers tied to it. If you do not have any of these identifiers attached, you will not be able to access the menu. By default, only admins with the `manage.admins` permission can set identifiers. The master admin can enable self-editing in **Settings → General**.

### Menu Features

- **Player Modes:** NoClip, God Mode, SuperJump
- **Vehicle:** Spawn, fix, delete, boost
- **Teleport:** To waypoint, coordinates, or other players
- **Player Actions:** Bring, go to, spectate, freeze, heal (self, all, or configurable radius)
- **Live Spectate:** Watch a player's screen in real-time from the web panel (~5 FPS WebP stream)
- **Screenshot Capture:** Built-in player screenshot via NUI (no `screenshot-basic` needed)
- **Punishments:** Ban, warn, kick with revocation support and editable ban durations
- **Reports:** Player-initiated tickets (Player Report, Bug Report, Question) with admin review workflow
- **Player List:** Real-time list with search and sort by distance/ID/name
- **Overhead IDs:** Player name tags with color-coded tags above heads
- **Troll Actions:** Drunk effect, set fire, wild attack

---

## ConVars

These ConVars control the behavior of the in-game menu, and are configured through the fxPanel Settings page. ConVars configured in the settings page should not be set manually.

### Settings Page ConVars

**`txAdmin-menuEnabled`**
Enable or disable the in-game menu entirely. Changing it requires a server restart.
Default: `true`

**`txAdmin-menuAlignRight`**
Align the menu to the right side of the screen instead of the left.
Default: `false`

**`txAdmin-menuPageKey`**
Change the key used to switch pages within the menu. The value must be the exact browser key code (use [keycode.info](https://keycode.info/) and the `event.code` value).
Default: `Tab`

**`txAdmin-playerModePtfx`**
Play particle effects and sound when toggling player modes (NoClip, God Mode, etc.).
Default: `true`

**`txAdmin-hideAdminInPunishments`**
Never show the admin name in ban/warn messages shown to players.
Default: `true`

**`txAdmin-hideAdminInMessages`**
Do not show the admin name on announcements or DMs.
Default: `false`

**`txAdmin-hideDefaultAnnouncement`**
Suppress the default announcement display, allowing you to implement your own via the `txAdmin:events:announcement` event.
Default: `false`

**`txAdmin-hideDefaultDirectMessage`**
Suppress the default direct message display, allowing you to implement your own via the `txAdmin:events:playerDirectMessage` event.
Default: `false`

**`txAdmin-hideDefaultWarning`**
Suppress the default warning display, allowing you to implement your own via the `txAdmin:events:playerWarned` event.
Default: `false`

**`txAdmin-hideDefaultScheduledRestartWarning`**
Suppress the default scheduled restart warning display, allowing you to implement your own via the `txAdmin:events:scheduledRestart` event.
Default: `false`

### ConVar Only (not in Settings page)

These ConVars are set directly in your `server.cfg` using `setr`.

**`txAdmin-debugMode`**
Toggle debug printing on the server and client.
Default: `false`
Usage: `setr txAdmin-debugMode true`

**`txAdmin-menuPlayerIdDistance`**
The distance at which overhead player IDs become visible (if toggled on). The game engine limits tags to ~300m, so values above that are ineffective.
Default: `150`
Usage: `setr txAdmin-menuPlayerIdDistance 100`

**`txAdmin-menuDrunkDuration`**
How many seconds the drunk effect (troll action) should last.
Default: `30`
Usage: `setr txAdmin-menuDrunkDuration 120`

**`txAdmin-menuAnnounceNotiPos`**
Position of the fxPanel announcement notification. Must be one of: `top-center`, `top-left`, `top-right`, `bottom-center`, `bottom-left`, `bottom-right`.
Default: `top-center`
Usage: `set txAdmin-menuAnnounceNotiPos top-right`

---

## Chat Commands

In addition to the menu, these chat commands are available:

**`/tx`** or **`/txadmin`**
Open the admin menu. Optionally pass a player ID: `/tx 42`

**`/tpm`**
Teleport to the waypoint set on your map.

**`/goto <id>`**
Teleport to a player by their server ID.

**`/txAdmin-reauth`**
Retrigger the authentication process. Available to all players (no admin required).

---

## Troubleshooting

**Nothing happens when typing /tx**
Your menu is probably disabled. Check the `txAdmin-menuEnabled` ConVar.

**Red authentication error message**
If you are registered on fxPanel, type `/txAdmin-reauth` in the chat to retry authentication.

**"Invalid Request: source" error**
This means the source IP of the HTTP request from FXServer to fxPanel is not a localhost address, which can happen if your host has multiple IPs. To disable this protection, edit your `config.json` and set `webServer.disableNuiSourceCheck` to `true`, then restart fxPanel.
