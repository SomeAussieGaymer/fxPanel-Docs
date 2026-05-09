# Development

Project structure, dev setup, and building from source.

---

## Project Structure

fxPanel is a monorepo using npm workspaces with the following top-level packages:

**`core/`** — Node.js backend (TypeScript + esbuild)
Boot process, deployer, modules (AdminStore, Database, FxMonitor, etc.), web routes

**`panel/`** — Main React web UI (Vite + Tailwind CSS v4)
Dashboard, player management, console, resources, settings

**`nui/`** — React in-game NUI menu (Vite + Tailwind CSS v4)
Player list, admin tools, vehicle spawn, spectate

**`resource/`** — FiveM/RedM resource (Lua 5.4 + JavaScript)
Server/client scripts for logging, admin tools, reports

**`shared/`** — Cross-workspace TypeScript types & utilities
Branded types, API types, permissions, helpers

**`scripts/`** — Development and build scripts
Lint, typecheck, build workflows

---

## Dev Setup

Prerequisites:

- **Windows** — the builder doesn't work on other operating systems
- Node.js **v22.9** or newer
- A working FXServer installation

1. Clone the repository **outside** your FXServer directory.

2. Install dependencies:
   ```
   npm install
   ```

3. Prepare the project:
   ```
   npm run prepare
   ```

4. Create a `.env` file in the root with your FXServer path:
   ```
   TXDEV_FXSERVER_PATH=/path/to/your/fxserver
   ```

---

## Workflows

### Core + Panel Development

Run two terminals side by side:

```bash
# Terminal 1 — Panel (hot reload)
cd panel && npm run dev
```

```bash
# Terminal 2 — Core (watches, rebuilds, restarts FXServer)
cd core && npm run dev
```

### NUI Menu Development

The NUI menu can be developed in two modes:

```bash
# Game mode — renders inside FiveM
cd nui && npm run dev
```

```bash
# Browser mode — standalone in your browser
cd nui && npm run browser
```

> ⚠️ **Important:** For every change you will need to restart the `monitor` resource. Unless you started the server with `+setr txAdmin-debugMode true`, fxPanel will detect the restart as a crash and restart your server. In game mode, the Vite builder takes 10–30 seconds to finish before you can restart the resource.

---

## Testing & Building

**`npm run test --workspaces`**
Run all tests across workspaces using Vitest.

**`npm run typecheck -w core`**
Run TypeScript type checking for the core package.

**`npm run lint -w core`**
Run ESLint for the core package.

**`npm run build`**
Build all packages for production. This creates a deployable `monitor/` folder.