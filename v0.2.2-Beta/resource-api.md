# Resource Exports

Use server-side exports to integrate your resources with fxPanel's moderation and admin systems.

> All exports are called from server-side Lua via `exports['monitor']:exportName(...)`. Every action export requires an authenticated fxPanel admin as the source and enforces the appropriate permission check. Actions are fully logged in fxPanel's action history.

> ⛔ **Important:** The `serverId` parameter in action exports refers to the **admin** performing the action, not the target. The admin must be authenticated with fxPanel and have the required permission, otherwise the call will silently fail with a server-console error.

---

## Permission & Admin Info

Query whether a player is an fxPanel admin and inspect their permissions.

### `hasPermission(serverId, permission)`

Check if a player has a specific fxPanel permission. Returns `true` if the player is an admin with the given permission (or `all_permissions`).

```lua
-- Parameters
serverId   : number  -- The player's server ID
permission : string  -- Permission to check (e.g. 'players.ban', 'players.kick')

-- Returns
boolean  -- true if admin has the permission

-- Example
local canBan = exports['monitor']:hasPermission(source, 'players.ban')
if canBan then
    print('Player can ban!')
end
```

### `isPlayerAdmin(serverId)`

Check if a player is an authenticated fxPanel admin, regardless of their permissions.

```lua
-- Parameters
serverId : number  -- The player's server ID

-- Returns
boolean  -- true if player is an admin

-- Example
if exports['monitor']:isPlayerAdmin(source) then
    print('Player is an fxPanel admin')
end
```

### `getAdminUsername(serverId)`

Get the fxPanel username of an admin. Returns `nil` if the player is not an admin.

```lua
-- Parameters
serverId : number  -- The player's server ID

-- Returns
string | nil  -- The admin's fxPanel username, or nil

-- Example
local name = exports['monitor']:getAdminUsername(source)
if name then
    print('Admin username: ' .. name)
end
```

### `getAdminPermissions(serverId)`

Get the full list of permission strings for an admin. Returns `nil` if the player is not an admin.

```lua
-- Parameters
serverId : number  -- The player's server ID

-- Returns
table | nil  -- Array of permission strings, or nil

-- Example
local perms = exports['monitor']:getAdminPermissions(source)
if perms then
    for _, perm in ipairs(perms) do
        print('Permission: ' .. perm)
    end
end
```

---

## Moderation Actions

Perform moderation actions through fxPanel. All actions are logged, trigger the corresponding events, and respect the admin's permissions.

### `kickPlayer(serverId, targetId, reason)`

Kick a player through fxPanel. Requires `players.kick` permission. The kick is logged in the action history and fires the `txAdmin:events:playerKicked` event.

```lua
-- Parameters
serverId : number       -- The admin's server ID
targetId : number       -- The target player's server ID
reason   : string | nil -- Kick reason (defaults to 'no reason provided')

-- Example
exports['monitor']:kickPlayer(source, targetPlayerId, 'Breaking server rules')
```

### `banPlayer(serverId, targetId, reason, duration)`

Ban a player through fxPanel. Requires `players.ban` permission. The ban is logged in the action history and fires the `txAdmin:events:playerBanned` event.

```lua
-- Parameters
serverId : number       -- The admin's server ID
targetId : number       -- The target player's server ID
reason   : string | nil -- Ban reason (defaults to 'no reason provided')
duration : string | nil -- Duration string (defaults to 'permanent')
                        -- Examples: '2 hours', '1 day', '1 week', 'permanent'

-- Examples
-- Permanent ban
exports['monitor']:banPlayer(source, targetPlayerId, 'Cheating')

-- Temporary ban
exports['monitor']:banPlayer(source, targetPlayerId, 'Toxicity', '2 hours')

-- 1-week ban
exports['monitor']:banPlayer(source, targetPlayerId, 'Repeated offenses', '1 week')
```

### `warnPlayer(serverId, targetId, reason)`

Warn a player through fxPanel. Requires `players.warn` permission. The warning is logged in the action history and fires the `txAdmin:events:playerWarned` event.

```lua
-- Parameters
serverId : number       -- The admin's server ID
targetId : number       -- The target player's server ID
reason   : string | nil -- Warn reason (defaults to 'no reason provided')

-- Example
exports['monitor']:warnPlayer(source, targetPlayerId, 'Please read the server rules')
```

### `sendAnnouncement(serverId, message)`

Send a server-wide announcement through fxPanel. Requires `announcement` permission. The announcement is displayed to all players and fires the `txAdmin:events:announcement` event.

```lua
-- Parameters
serverId : number -- The admin's server ID
message  : string -- The announcement message

-- Example
exports['monitor']:sendAnnouncement(source, 'Server restarting in 5 minutes!')
```

---

## Permission Strings

Reference of permission strings used with `hasPermission()` and enforced by the action exports.

```lua
-- Moderation
players.ban          -- Ban players
players.kick         -- Kick players
players.warn         -- Warn players
players.unban        -- Revoke bans
players.direct_message -- DM players
players.whitelist    -- Whitelist players

-- Server
control.server       -- Start/stop/restart server
announcement         -- Send announcements
console.view         -- View console
console.write        -- Write to console
commands.resources   -- Start/stop resources
server.cfg.editor    -- Edit server.cfg

-- Admin
all_permissions      -- Root permission (includes all)
manage.admins        -- Manage admin accounts
settings.view        -- View settings
settings.write       -- Change settings
txadmin.log.view     -- View logs

-- In-Game Menu
menu.vehicle.spawn   -- Spawn vehicles
menu.vehicle.fix     -- Fix vehicles
menu.vehicle.boost   -- Boost vehicles
menu.vehicle.delete  -- Delete vehicles
menu.clear_area      -- Clear area
menu.viewids         -- View player IDs overhead

-- Player Actions
players.freeze       -- Freeze players
players.heal         -- Heal players
players.noclip       -- NoClip mode
players.godmode      -- God mode
players.superjump    -- Super jump
players.spectate     -- Spectate players
players.teleport     -- Teleport
players.troll        -- Troll actions
players.reports      -- View reports
players.delete       -- Delete player data
```
