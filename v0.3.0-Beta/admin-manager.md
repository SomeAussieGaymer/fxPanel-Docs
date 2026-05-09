# Admin Manager

The Admin Manager is the staff administration surface for fxPanel. It is available at `/admins` and is guarded by the `manage.admins` permission in the panel router and backend routes.

Use it to:

- Create, edit, delete, and reset passwords for admin accounts.
- Assign granular permissions or `all_permissions`.
- Save and apply reusable permission presets.
- Review per-admin activity stats and recent action counts.
- Bulk apply permissions for selected non-master admins, with the implementation constraint noted below.

## Architecture

The React page lives in `panel/src/pages/AdminManager`. It reads data through authenticated JSON endpoints and uses SWR for list, stats, and preset refreshes.

| Area | Backend route | Purpose |
| ---- | ------------- | ------- |
| Admin list | `GET /adminManager/list` | Returns admin identity, permissions, online state, and current-account marker. |
| Admin stats | `GET /adminManager/stats` | Returns per-admin ban, warn, kick, revoked action, total action, and ticket resolution counts. |
| Recent actions | `GET /adminManager/adminActions?admin=<name>` | Returns recent ban, warn, and kick actions for one admin. |
| Add/edit/delete/reset | `POST /adminManager/:action` | Handles `add`, `edit`, `delete`, and `resetPassword`. |
| Presets | `GET /adminManager/presets`, `POST /adminManager/presets` | Loads and saves the full permission preset list. |

Permission presets are stored in `permissionPresets.json` under the configured txData path. Saving presets writes the full array, then triggers websocket auth re-checks.

## Permission Model

Permissions are defined centrally in `shared/permissions.ts`. Categories are `System`, `Server`, `In-Game Menu`, `Player Management`, and `Addons`.

Important rules:

- `master` accounts pass every permission check.
- `all_permissions` grants every registered permission.
- Admins without `master` or `all_permissions` cannot grant permissions they do not already have.
- Some permissions are marked `dangerous`; the panel asks for confirmation before granting newly-added dangerous permissions.
- Legacy combined permissions are migrated in code: `menu.vehicle` is split into vehicle spawn/fix/boost/delete, and `players.playermode` is split into noclip/godmode/superjump.

Example permission preset:

```json
{
    "id": "custom:moderator",
    "name": "Moderator",
    "permissions": ["console.view", "players.warn", "players.kick", "players.reports"]
}
```

## Account Constraints

Admin usernames are validated by the FiveM username regex used by the backend. They must be 3-20 characters, start and end with a letter, number, or underscore, and may contain letters, numbers, underscores, dots, and hyphens.

Cfx.re identity accepts either:

- A game identifier such as `fivem:123456`.
- A Cfx.re username that can be resolved through the Cfx.re forum API.

Discord identity accepts a Discord snowflake ID and is stored as a `discord:<id>` provider identifier.

New admins receive a generated temporary password. The password is shown once and the admin is forced to change it on first login. Password reset uses the same temporary-password flow.

## Safety Rules

- You cannot edit yourself from the Admin Manager.
- You cannot delete yourself.
- Non-master admins cannot edit or reset passwords for master admins.
- Master admins cannot be deleted.
- Bulk permission apply skips master admins and the current account.
- Bulk permission apply replaces the selected admins' permission arrays; it is not an additive merge.
- In the current implementation, bulk permission apply sends an empty Discord ID through the edit route, so it removes the Discord provider from each eligible target. Re-add Discord IDs after bulk changes if those accounts need Discord login or role sync.

## Usage

1. Open `/admins`.
2. Use **Add Admin** to create an admin with a username, optional Cfx.re ID, optional Discord ID, and permission set.
3. Copy the temporary password immediately; it is not shown again.
4. Use **Permission Presets** to save repeatable permission sets for common staff roles.
5. Use **Bulk Apply** only when you want to replace permissions for selected eligible admins.

For example, to create a support-only staff account, grant read-oriented permissions such as `console.view` and `players.reports`, then add action permissions like `players.warn` or `players.kick` only if that role should moderate players directly.
