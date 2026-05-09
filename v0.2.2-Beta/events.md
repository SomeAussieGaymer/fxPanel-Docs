# Events

Integrate your resources with fxPanel using server-side events.

> All events follow the pattern `txAdmin:events:<eventName>`. Listen for these in your server-side Lua or JavaScript resources.

> ⛔ **Important:** Do not fully rely on events where consistency is key, since they may fire while the server is stopped. For instance, a player could be whitelisted or banned while the server is offline, and your resource would not be notified.

---

## Server Events

### `txAdmin:events:announcement`

Fired when an announcement is made using fxPanel.

```lua
{
  author: string,       -- admin name or "fxPanel"
  message: string
}
```

### `txAdmin:events:serverShuttingDown`

Fired when the server is about to shut down. Can be triggered by a scheduled or unscheduled stop/restart, by an admin or by the system.

```lua
{
  delay: number,        -- ms until server process is killed
  author: string,       -- admin name or "fxPanel"
  message: string
}
```

### `txAdmin:events:scheduledRestart`

Fires at countdown intervals: **30, 15, 10, 5, 4, 3, 2, 1** minutes before a scheduled restart.

```lua
{
  secondsRemaining: number,
  translatedMessage: string
}
```

### `txAdmin:events:scheduledRestartSkipped`

Fired when an admin skips the next scheduled restart.

```lua
{
  secondsRemaining: number,
  temporary: boolean,   -- true if it was a temporary restart
  author: string
}
```

---

## Player Events

### `txAdmin:events:playerBanned`

Fired when a player is banned.

```lua
{
  author: string,
  reason: string,
  actionId: string,
  expiration: number | false,      -- timestamp or false if permanent
  durationInput: string,
  durationTranslated: string | nil,
  targetNetId: number | nil,       -- nil if ban applied to identifiers only
  targetIds: string[],
  targetHwids: string[],           -- may be empty array
  targetName: string,              -- "identifiers" if legacy ban
  kickMessage: string
}
```

### `txAdmin:events:playerWarned`

Fired when a player receives a warning.

```lua
{
  author: string,
  reason: string,
  actionId: string,
  targetNetId: number | nil,       -- nil if target is not online
  targetIds: string[],
  targetName: string
}
```

### `txAdmin:events:playerKicked`

Fired when a player is kicked. A target of `-1` means all players are being kicked.

```lua
{
  target: number,       -- player ID, or -1 if kicking everyone
  author: string,
  reason: string,
  dropMessage: string   -- translated message shown to the player
}
```

### `txAdmin:events:playerDirectMessage`

Fired when an admin sends a DM to a player.

```lua
{
  target: number,       -- player ID
  author: string,
  message: string
}
```

### `txAdmin:events:playerHealed`

Fired when a heal event is triggered. A target of `-1` means the entire server was healed.

```lua
{
  target: number,       -- player ID, or -1 for all
  author: string
}
```

---

## Whitelist Events

### `txAdmin:events:whitelistPlayer`

Fired when a player is whitelisted or has their whitelisted status revoked. Only fires for already-registered players, not for pending whitelist requests or pre-approvals.

```lua
{
  action: "added" | "removed",
  license: string,
  playerName: string,
  adminName: string
}
```

### `txAdmin:events:whitelistPreApproval`

Fired when an identifier is manually added to the whitelist pre-approvals. When a player with this identifier connects, they will be saved as whitelisted without triggering `whitelistPlayer`. Not fired when a whitelist request is approved (use `whitelistRequest` for that).

```lua
{
  action: "added" | "removed",
  identifier: string,          -- e.g. "discord:xxxxxx"
  playerName?: string,         -- absent when action is "removed"
  adminName: string
}
```

### `txAdmin:events:whitelistRequest`

Fired whenever an event related to a whitelist request occurs.

```lua
{
  action: "requested" | "approved" | "denied" | "deniedAll",
  playerName?: string,         -- absent when action is "deniedAll"
  requestId?: string,          -- e.g. "Rxxxx", absent when "deniedAll"
  license?: string,            -- absent when action is "deniedAll"
  adminName?: string           -- absent when action is "requested"
}
```

---

## Other Events

### `txAdmin:events:actionRevoked`

Fired when an admin revokes a database action (e.g. ban, warn).

```lua
{
  actionId: string,
  actionType: string,
  actionReason: string,
  actionAuthor: string,
  playerName: string | false,  -- false if not applicable
  playerIds: string[],
  playerHwids: string[],       -- may be empty array
  revokedBy: string
}
```

### `txAdmin:events:adminAuth`

Fired when an admin authenticates in-game or loses admin permissions. Useful for anti-cheats to ignore fxPanel admins.

```lua
{
  netid: number,               -- player ID, or -1 for forced reauth of all
  isAdmin: boolean,
  username?: string            -- fxPanel username, present on auth
}
```

### `txAdmin:events:adminsUpdated`

Fired when the admin list changes (additions, removals, permission or identifier changes). Used by the fxPanel resource to force admins to refresh their auth.

```lua
number[]  -- array of NetIds of online admins
```

### `txAdmin:events:configChanged`

Fired when fxPanel settings are saved. This can be used by resources to react to configuration changes without requiring a restart.

```lua
-- No payload (empty event)
```
