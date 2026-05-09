# Permissions

Granular permission system with individual permissions and built-in presets.

---

## Overview

fxPanel uses a non-combined permission system — each permission is granted individually rather than being grouped into a single role string. This gives server owners fine-grained control over what each admin can do.

Admins can be managed through the Admin Manager in the web panel. Each admin account can be linked to Discord or Cfx.re identifiers for in-game authentication, and optionally protected with Two-Factor Authentication (TOTP). Permissions are saved in the `txData/admins.json` file.

---

## Permission List

### System

- **`all_permissions`** — Root permission — grants every other permission. Use with caution.
- **`manage.admins`** — Create, edit, and delete admin accounts.
- **`settings.view`** — View settings (sensitive tokens hidden).
- **`settings.write`** — Modify fxPanel settings.
- **`txadmin.log.view`** — View fxPanel system and admin action logs.

### Server

- **`console.view`** — View the live FXServer console.
- **`console.write`** — Execute commands in the FXServer console.
- **`control.server`** — Start, stop, and restart the FXServer process and scheduler.
- **`announcement`** — Broadcast announcements to all players.
- **`commands.resources`** — Start or stop server resources.
- **`server.cfg.editor`** — Read and write the server.cfg file.
- **`server.log.view`** — View FXServer log output.

### In-Game Menu

- **`menu.vehicle.spawn`** — Spawn vehicles via the in-game menu.
- **`menu.vehicle.fix`** — Repair vehicles via the in-game menu.
- **`menu.vehicle.boost`** — Boost vehicles via the in-game menu.
- **`menu.vehicle.delete`** — Delete vehicles via the in-game menu.
- **`menu.clear_area`** — Reset a world area via the in-game menu.
- **`menu.viewids`** — See player IDs overhead in-game.

### Player Management

- **`players.direct_message`** — Send direct messages to players.
- **`players.whitelist`** — Whitelist players.
- **`players.warn`** — Issue warnings to players.
- **`players.kick`** — Kick players from the server.
- **`players.ban`** — Ban players from the server.
- **`players.unban`** — Revoke existing player bans.
- **`players.freeze`** — Freeze player peds in-game.
- **`players.heal`** — Heal self or all players.
- **`players.noclip`** — Toggle NoClip mode for yourself.
- **`players.godmode`** — Toggle invincibility for yourself.
- **`players.superjump`** — Toggle super jump for yourself.
- **`players.spectate`** — Spectate players.
- **`players.teleport`** — Teleport self or bring/go to players.
- **`players.troll`** — Use the troll menu on players.
- **`players.reports`** — View and manage player reports.
- **`players.delete`** — Delete bans/warns, players, and player identifiers (dangerous).

---

## Presets

Permission presets allow you to create reusable groups of permissions that can be quickly applied when adding or editing admin accounts via the Admin Manager.

**Full Admin** — Grants `all_permissions`.

**Moderator** — Console view, server log view, direct message, warn, kick, ban, freeze, spectate, teleport, and view player IDs.

**Supporter** — Console view, direct message, warn, kick, spectate, and view player IDs.

You can also create, name, edit, and delete your own custom presets through the Admin Manager page.
