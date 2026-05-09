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

- **Operating system**
    - **Core dev workflow** (`cd core && npm run dev`) — Windows or Linux. The dev runner spawns the FXServer binary directly (`FXServer.exe` on Windows, `FXServer` / `run.sh` on Linux).
    - **Watch-only dev workflow** (`cd core && npm run dev:watch-only`) — Windows, Linux, **and macOS**. Rebuilds + syncs files into a target `monitor/` folder but does not spawn FXServer. Intended for macOS or for any setup where FXServer runs on another machine / inside Docker. See [Watch-only dev mode](#watch-only-dev-mode) below.
    - **Production build** (`npm run build`) — Windows, Linux, **and macOS**. The builder is plain Node + esbuild and has no platform-specific code paths, so the resulting `monitor/` folder is identical regardless of the host OS.
- Node.js **v22.9** or newer
- A working FXServer installation (only required for the dev workflows, not for `npm run build`)

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

Common `TXDEV_*` variables:

| Variable | Required when | Default / behavior |
| -------- | ------------- | ------------------ |
| `TXDEV_FXSERVER_PATH` | Core dev and NUI game mode | Must point at an FXServer root containing `citizen/system_resources/monitor`; full dev mode also requires `FXServer.exe`, `FXServer`, or `run.sh`. |
| `TXDEV_VITE_URL` | Core runtime when `TXDEV_ENABLED=true` | `http://localhost:40122` in the dev scripts. |
| `TXDEV_NO_SPAWN` | Optional for core dev | Any set value enables watch-only mode; macOS enables this automatically. |
| `TXDEV_LAUNCH_ARGS` | Optional for core dev | Whitespace-split arguments passed to the spawned FXServer process. |
| `TXDEV_CFXKEY`, `TXDEV_STEAMKEY`, `TXDEV_EXT_STATS_HOST` | Optional dev helpers | Forwarded through the dev environment when set. |

When `cd core && npm run dev` spawns FXServer, it injects `TXDEV_ENABLED=true` and `TXDEV_SRC_PATH=<repo root>` into the child process. If you run FXServer yourself in watch-only mode and still want the panel served from Vite, set `TXDEV_ENABLED=true`, `TXDEV_SRC_PATH` to the repository root, and `TXDEV_VITE_URL` to the panel dev URL in that FXServer environment.

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

**Public addon server note:** If any running addon enables `publicRoutes: true` (for example `addons/react-site`), the addon public server is currently HTTPS-only. Local dev requires `TXHOST_ADDON_PUBLIC_TLS_KEY` + `TXHOST_ADDON_PUBLIC_TLS_CERT` or `TXHOST_ADDON_PUBLIC_TLS_KEY_FILE` + `TXHOST_ADDON_PUBLIC_TLS_CERT_FILE`. Without them, core will log that the public server could not start.

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

**Important:** For every change you will need to restart the `monitor` resource. Unless you started the server with `+setr txAdmin-debugMode true`, fxPanel will detect the restart as a crash and restart your server. In game mode, the Vite builder takes 10–30 seconds to finish before you can restart the resource.

**Clipboard note:** FiveM's NUI CEF can block `navigator.clipboard` through permissions policy. For copy flows in the NUI, use `nui/src/utils/copyToClipboard.ts` instead of calling the Clipboard API directly.

**Live Console note:** Terminal resizing should go through `FitAddon.fit()` and react to container and `window.visualViewport` changes, not only `window.resize`, otherwise the mobile layout can keep stale rows/columns.

### Watch-only dev mode

Use this when you can't (or don't want to) have the dev script spawn FXServer for you — for example on **macOS** (where there is no native FXServer build), against a **remote** server reached over SSH, or against a **Dockerized** Linux FXServer.

It does the same chokidar file sync + esbuild watch as `npm run dev`, but:

- Does **not** call `FXServer.exe` / `FXServer` / `run.sh`.
- Does **not** auto-restart the server on rebuild — you restart it yourself.
- Skips the binary check in `getFxsPaths`, so `TXDEV_FXSERVER_PATH` only has to point at a folder that contains `citizen/system_resources/monitor`.

Three equivalent ways to enable it:

```bash
# 1. Cross-platform npm script (recommended)
cd core && npm run dev:watch-only

# 2. POSIX shells (Linux / macOS)
cd core && TXDEV_NO_SPAWN=1 npm run dev

# 3. PowerShell (Windows)
$env:TXDEV_NO_SPAWN="1"; cd core; npm run dev
```

On macOS the dev script auto-detects the platform and turns watch-only mode on by itself, so `npm run dev` and `npm run dev:watch-only` behave identically there.

### Developing against a Dockerized FXServer

A common cross-platform setup: run FXServer inside a Linux container, bind-mount its data directory onto your host, and point fxPanel's watcher at the mount.

1. **Run a Linux FXServer container** with its data directory mounted somewhere on your host. Any FXServer image works; the only requirement is that the host can see `<mount>/citizen/system_resources/monitor`. Example using a generic image:

    ```bash
    docker run -d --name fxserver \
        -p 30120:30120/tcp -p 30120:30120/udp \
        -p 40120:40120/tcp \
        -v "$PWD/fxserver-data:/opt/cfx-server-data" \
        your/fxserver-image
    ```

2. **Point fxPanel at the mount** in your `.env` file:

    ```
    TXDEV_FXSERVER_PATH=/absolute/path/to/fxserver-data
    TXDEV_VITE_URL=http://localhost:40122
    ```

3. **Start the watch-only dev script** on your host:

    ```bash
    cd core && npm run dev:watch-only
    cd panel && npm run dev   # in a second terminal
    ```

4. **Restart FXServer yourself after rebuilds**, e.g.:

    ```bash
    docker restart fxserver
    # or, inside the container:
    docker exec fxserver pkill -HUP FXServer
    ```

    A common shortcut is to enable `+setr txAdmin-debugMode true` so that resource restarts triggered by fxPanel itself don't cause a full server restart loop.

This flow is the recommended way to develop fxPanel on macOS.

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

Supported on Windows, Linux, and macOS. From a clean checkout:

```bash
npm ci
npm run build
```

Output is written to `monitor/`. The core dev workflow can spawn FXServer on Windows and Linux; macOS and remote/container targets should use watch-only mode.

> **macOS note:** if `npm ci` fails with `Failed to load native binding` / `Cannot find module '@swc/core-darwin-arm64'`, you've hit [npm/cli#4828](https://github.com/npm/cli/issues/4828) — `npm ci` skips platform-specific optional deps when the committed lockfile was generated on a different OS. Use `npm install --include=optional` instead of `npm ci` on first checkout, or delete `node_modules` and re-run `npm install`.
