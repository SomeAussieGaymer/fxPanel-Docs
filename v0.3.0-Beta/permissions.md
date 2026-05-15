# Permissions

Granular permission system with individual permissions and custom presets.

---

## Overview

fxPanel uses a non-combined permission system — each permission is granted individually rather than being grouped into a single role string. This gives server owners fine-grained control over what each admin can do.

Admins can be managed through the Admin Manager in the web panel. Each admin account can be linked to Discord or Cfx.re identifiers for in-game authentication, and optionally protected with Two-Factor Authentication (TOTP). Permissions are saved in the `txData/admins.json` file. See [Admin Manager](/docs/v0.3.0-Beta/admin-manager) for account workflows, safety rules, and API behavior.

---

## Permission List

### System

| Permission | Description |
|------------|-------------|
| `all_permissions` | Root permission — grants every other permission. Use with caution. |
| `manage.admins` | Create, edit, and delete admin accounts. Also required for editing admin identifiers when self-editing is disabled. |
| `settings.view` | View settings (sensitive tokens hidden). |
| `settings.write` | Modify fxPanel settings. |
| `txadmin.log.view` | View fxPanel system and admin action logs. |

### Server

| Permission | Description |
|------------|-------------|
| `console.view` | View the live FXServer console. |
| `console.write` | Execute commands in the FXServer console. |
| `control.server` | Start, stop, and restart the FXServer process and scheduler. |
| `announcement` | Broadcast announcements to all players. |
| `commands.resources` | Start or stop server resources. |
| `server.cfg.editor` | Read and write the server.cfg file. |
| `server.log.view` | View FXServer log output. |

### In-Game Menu

| Permission | Description |
|------------|-------------|
| `menu.vehicle.spawn` | Spawn vehicles via the in-game menu. |
| `menu.vehicle.fix` | Repair vehicles via the in-game menu. |
| `menu.vehicle.boost` | Boost vehicles via the in-game menu. |
| `menu.vehicle.delete` | Delete vehicles via the in-game menu. |
| `menu.clear_area` | Reset a world area via the in-game menu. |
| `menu.viewids` | See player IDs overhead in-game. |

### Player Management

| Permission | Description |
|------------|-------------|
| `players.direct_message` | Send direct messages to players. |
| `players.whitelist` | Whitelist players. |
| `players.warn` | Issue warnings to players. |
| `players.kick` | Kick players from the server. |
| `players.ban` | Ban players from the server. |
| `players.unban` | Revoke existing player bans. |
| `players.freeze` | Freeze player peds in-game. |
| `players.heal` | Heal self or all players. |
| `players.noclip` | Toggle NoClip mode for yourself. |
| `players.godmode` | Toggle invincibility for yourself. |
| `players.superjump` | Toggle super jump for yourself. |
| `players.spectate` | Spectate players. |
| `players.teleport` | Teleport self or bring/go to players. |
| `players.troll` | Use the troll menu on players. |
| `players.reports` | View and manage player reports. |
| `manage_tickets` | Delete any ticket and exclude closed tickets from automatic retention cleanup (dangerous). |
| `players.delete` | Delete bans/warns, players, and player identifiers (dangerous). |

---

## Permission Presets

Permission presets let you save reusable groups of permissions that can be quickly applied when adding or editing admin accounts via the Admin Manager.

Presets are fully custom. You can create, name, edit, and delete them through the Admin Manager page to match your own staff structure.

Presets are stored in `permissionPresets.json` under the configured txData path. Saving presets replaces the full preset list, so export or back up this file before large permission reorganizations.

Example:

```json
{
    "id": "custom:moderator",
    "name": "Moderator",
    "permissions": ["console.view", "players.warn", "players.kick", "players.reports"]
}
```

---

## Permission Migrations

fxPanel migrates older combined permission IDs into newer granular permissions when admin data is loaded:

| Old permission | New permissions |
| -------------- | --------------- |
| `menu.vehicle` | `menu.vehicle.spawn`, `menu.vehicle.fix`, `menu.vehicle.boost`, `menu.vehicle.delete` |
| `players.playermode` | `players.noclip`, `players.godmode`, `players.superjump` |

`players.ban` now only grants ban creation. Use `players.unban` separately for revoking bans.

### Discord Role Mappings

In **Settings -> Discord Bot -> Role Permission Mapping**, you can link one or more Discord role IDs to a permission preset.

For linked admins, fxPanel resolves their Discord guild roles during authentication and syncs every matched preset onto the admin account itself.

- Multiple matching mappings are merged together.
- Manually assigned permissions remain on the account.
- Only the Discord-synced slice is replaced when their roles change.

Because the synced permissions are written onto the admin record, staff can see those permissions in Admin Manager instead of relying on a hidden runtime-only permission overlay.

---

## Addon Permissions

Addons can define custom permission strings that appear alongside built-in permissions in the admin role editor. See [Addon Development → Custom Admin Permissions](/docs/v0.3.0-Beta/addon-development#custom-admin-permissions) for details.

---

## Identifier Editing

By default, admins **cannot** edit their own FiveM or Discord identifiers from the Account dialog. Only admins with the `manage.admins` permission can change identifiers.

The **master admin** can enable the `Allow Self Identifier Edit` toggle in **Settings → General** to let all admins change their own identifiers. When this setting is disabled, the Identifiers tab will only be visible to admins with the `manage.admins` permission.
