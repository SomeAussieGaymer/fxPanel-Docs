# fxPanel v0.3.0 Beta Docs

This folder contains source-verified notes for the v0.3.0 beta documentation set. It extends the existing project docs with areas that changed in the current codebase and need operator or maintainer guidance.

## Start Here

| Page | Covers |
| ---- | ------ |
| [Getting Started](getting-started.md) | Installation, first launch, and initial server setup. |
| [Configuration](configuration.md) | Environment variables, networking, provider branding, and defaults. |
| [Permissions](permissions.md) | Built-in permissions, presets, migrations, Discord role mappings, and identifier editing. |
| [Admin Manager](admin-manager.md) | Admin accounts, permission presets, staff stats, and backend constraints. |
| [Artifact Updater](artifact-updater.md) | FXServer artifact discovery, custom downloads, update phases, and platform-specific apply behavior. |

## Additional Guides

| Page | Covers |
| ---- | ------ |
| [In-Game Menu](in-game-menu.md) | Admin menu usage and in-game controls. |
| [Discord](discord.md) | OAuth login, bot integration, status embeds, logging, and slash commands. |
| [Addons](addon-development.md) | Addon manifests, APIs, panel/NUI integration, storage, routes, and lifecycle. |
| [Resource API](resource-api.md) | Resource exports and integration points. |
| [Events](events.md) | Event hooks emitted by fxPanel. |
| [Recipes](recipes.md) | Server deployer recipes. |
| [Logging](logging.md) | System and server log behavior. |
| [Development](development.md) | Monorepo structure, dev workflows, testing, and builds. |
| [Troubleshooting](troubleshooting.md) | Common installation, login, Discord, addon, and menu issues. |
| [Documentation Changes](documentation-changes-2026-05-10.md) | What was added or fixed in this documentation pass and why. |

## Source of Truth

These docs were checked against the current source in `fxPanel-v1.0.0` rather than inferred from the UI labels alone. When behavior changes, update the docs beside the relevant code change.

Key source areas:

- `panel/src/layout/MainRouter.tsx` for panel routes and permissions.
- `core/modules/WebServer/router.ts` for backend API paths and rate-limited groups.
- `shared/permissions.ts` for permission IDs, categories, dangerous flags, and migration notes.
- `core/routes/adminManager/*` and `panel/src/pages/AdminManager/*` for admin management behavior.
- `core/routes/fxserver/update*.ts`, `core/modules/FxUpdater/index.ts`, and `panel/src/pages/FxUpdater/FxUpdaterPage.tsx` for artifact updater behavior.

## Documentation Rules

- Document behavior that is verified in source.
- Prefer short sections, examples, and explicit constraints over broad feature lists.
- Call out destructive operations and permission requirements.
- Keep version-specific details in this folder until they are promoted to the main docs.
