# Documentation Changes - 2026-05-10

## What Changed

Audited the v0.3.0 beta documentation set under `docs/v0.3.0-Beta` against the current source in `fxPanel-v1.0.0` and corrected source mismatches found during the pass:

- `getting-started.md` now reflects the JSON-backed player/action database, current locale count, Linux launch command, and safer existing-installation backup wording.
- `configuration.md` now distinguishes production `process.env` handling from dev `.env` loading and records current `TXHOST_*` validation constraints.
- `development.md` now points to the actual core dev scripts, `TXDEV_*` variables, watch-only mode behavior, and platform constraints.
- `admin-manager.md` documents current bulk permission replacement behavior and the Discord identifier side effect in the edit route.
- `artifact-updater.md` now describes the validated background download route, asynchronous busy failures, and persisted failure marker behavior.
- `in-game-menu.md` now matches the registered keybind label, ConVar usage, available chat commands, and command permissions.
- `discord.md` now separates role-mapped authorization from moderation commands that still require a linked fxPanel admin account.
- `resource-api.md` now separates moderation exports from player tag exports and clarifies which exports enforce admin permissions.
- `logging.md` now describes the actual `monitor` resource logger and deprecated compatibility events instead of a non-existent `txAdmin:events:serverLog` API.
- `recipes.md` now matches recipe meta fields, context variables, `serverEndpoints`, `write_file`, and final `replace_string` interpolation behavior.
- `translation.md` now records the 35 shipped locale files and the custom locale reload/runtime copy behavior.
- `addon-development.md` now matches the current addon manifest schema, runtime modes, public HTTPS server behavior, public route limitations, addon permissions, hot-reload, and public port constraints.
- `troubleshooting.md` now removes stale permission and addon route assumptions and aligns addon/runtime guidance with the source.

## Why

The docs had several shipped-behavior claims that were either stale or too broad. The audit corrected those claims so operators and addon authors do not rely on behavior that the source does not implement:

- Production configuration reads environment variables from the host process and has stricter validation than the docs previously showed.
- Public addon routes are served only by the dedicated HTTPS public addon server; the main panel origin intentionally does not expose a `/site/<addonId>/` fallback.
- The addon permission schema currently accepts only `storage`, `players.read`, `players.write`, and `ws.push`; moderation/server/database/http permissions were not valid addon manifest permissions.
- The Server Log page is driven by built-in resource logger events and deprecated compatibility events, not a general custom `txAdmin:events:serverLog` API.
- Recipe `write_file` writes literal data; placeholder expansion is handled by `replace_string` and the deployer's final `server.cfg` replacement pass.

## Verification

The docs were verified against these source areas in `fxPanel-v1.0.0`:

- `panel/src/layout/MainRouter.tsx`
- `panel/src/pages/AdminManager/AdminManagerPage.tsx`
- `panel/src/pages/AdminManager/AdminEditDialog.tsx`
- `panel/src/pages/FxUpdater/FxUpdaterPage.tsx`
- `panel/src/pages/LiveConsole/LiveConsolePage.tsx`
- `panel/src/pages/ServerLog`
- `panel/src/hooks/fetch.ts`
- `panel/src/hooks/auth.ts`
- `core/modules/WebServer/router.ts`
- `core/modules/WebServer/getReactIndex.ts`
- `core/modules/AdminStore/index.ts`
- `core/modules/AdminStore/permissionPresets.ts`
- `core/modules/FxUpdater/index.ts`
- `core/modules/AddonManager`
- `core/modules/DiscordBot`
- `core/modules/Translator.ts`
- `core/modules/ConfigStore/schema`
- `core/modules/FxPlayerlist/index.ts`
- `core/routes/adminManager/actions.ts`
- `core/routes/adminManager/list.ts`
- `core/routes/adminManager/presets.ts`
- `core/routes/adminManager/stats.ts`
- `core/routes/addons.ts`
- `core/routes/authentication/changeIdentifiers.ts`
- `core/routes/deployer/actions.ts`
- `core/routes/fxserver/commands.ts`
- `core/routes/fxserver/updateStatus.ts`
- `core/routes/fxserver/updateDownload.ts`
- `core/routes/fxserver/updateApply.ts`
- `core/routes/history/actions.ts`
- `core/routes/player/actions.ts`
- `core/routes/whitelist/actions.ts`
- `core/deployer`
- `addon-sdk/src`
- `bot/commands/_fxpanel`
- `shared/adminApiTypes.ts`
- `shared/addonTypes.ts`
- `shared/localeMap.ts`
- `shared/otherTypes.ts`
- `shared/permissions.ts`
- `shared/consts.ts`
- `shared/txDevEnv.ts`
- `scripts/build`
- `resource/sv_main.lua`
- `resource/sv_admins.lua`
- `resource/sv_logger.lua`
- `resource/menu/client`
- `resource/menu/server/sv_main_page.lua`
- `resource/menu/server/sv_vehicle.lua`
- `resource/menu/client/cl_player_ids.lua`
- `resource/menu/client/cl_player_mode.lua`

## Scope

No source code was changed. This pass only edited or added markdown inside `docs/v0.3.0-Beta` as requested.
