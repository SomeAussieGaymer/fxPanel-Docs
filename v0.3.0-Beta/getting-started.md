# Getting Started

Install fxPanel and get your first server up and running.

---

## Introduction

fxPanel is a modern, full-featured web panel and in-game menu for managing FiveM and RedM servers. It provides real-time monitoring, player management, analytics, a comprehensive permission system, Discord integration, and an automated deployment system from a single interface.

fxPanel replaces the need for external player/action databases by default, using a self-contained JSON-backed database in the active `txData` profile. It ships with 35 locale files, custom theming support, and a recipe-based deployer for setting up new servers.

---

## Installation

fxPanel replaces the default txAdmin monitor that ships with FXServer. Installation steps differ slightly by platform.

### Windows

1. **Install FXServer** — Download and install any FXServer artifact from the [official Cfx.re builds page](https://runtime.fivem.net/artifacts/fivem/build_server_windows/master/).

2. **Delete the default monitor** — Navigate to `citizen\system_resources` inside the folder where you installed the FXServer artifact, and delete the `monitor` folder.

3. **Download fxPanel** — Go to the [fxPanel GitHub releases page](https://github.com/SomeAussieGaymer/fxPanel/releases) and download the latest release.

4. **Install fxPanel** — Drag and drop the downloaded fxPanel folder into `citizen\system_resources` (the same location you deleted the monitor folder from).

5. **Start FXServer** — Run `FXServer.exe`. fxPanel will automatically initialize with a profile in the `txData` folder.

6. Open your browser and navigate to `http://localhost:40120` (default port).

> ⛔ **Important:** Make sure you fully delete the existing `monitor` folder before adding fxPanel. Running both at the same time will cause conflicts.

> **Note:** The default port is `40120`. You can change it via the `TXHOST_TXA_PORT` environment variable. See [Configuration](/docs/v0.3.0-Beta/configuration) for details.

### Linux

1. **Install FXServer** — Download the Linux FXServer artifact and extract it to your desired location.

2. **Delete the default monitor** — Navigate to `citizen/system_resources` inside the FXServer directory and delete the `monitor` folder:
   ```bash
   rm -rf /path/to/fxserver/citizen/system_resources/monitor
   ```

3. **Download fxPanel** — Download the latest release from the [fxPanel GitHub releases page](https://github.com/SomeAussieGaymer/fxPanel/releases).

4. **Install fxPanel** — Extract the fxPanel folder into `citizen/system_resources`:
   ```bash
   cp -r fxPanel-v1.0.0/monitor /path/to/fxserver/citizen/system_resources/
   ```

5. **Start FXServer** — Run the server start script without `+exec server.cfg`. fxPanel will automatically initialize with a profile in the `txData` folder:
   ```bash
   ./run.sh
   ```

6. Open your browser and navigate to `http://your-server-ip:40120`.

> **Note:** Ensure the `monitor` directory has the correct ownership and permissions for the user running FXServer.

### Existing Installations

The same steps above can be followed to install fxPanel on servers that already have txAdmin or a previous version of fxPanel installed. Stop FXServer, back up `txData`, delete the existing `monitor` folder, and replace it with the latest fxPanel release.

> **Backup note:** Existing profiles, player database files, admin accounts, and configuration live under `txData`. fxPanel is designed to reuse that data, but backing it up before replacing `monitor` gives you a rollback point if installation is interrupted or the wrong folder is replaced.

---

## Quick Start

After installation, follow these steps to get your server running:

1. **Create a Master Account** — On first launch, you'll be prompted to create a master admin account with full permissions.

2. **Deploy a Server** — Use the built-in Deployer with a recipe to set up a server automatically, or point fxPanel at an existing server data folder.

3. **Configure Settings** — Visit the Settings page to configure your server name, CFX license key, database connection, Discord bot, and more.

4. **Start the Server** — Click the Start button on the dashboard. fxPanel will manage the FXServer process with automatic crash/hang recovery.

5. **Access In-Game** — Join your server and type `/tx` to open the admin menu. See [In-Game Menu](/docs/v0.3.0-Beta/in-game-menu) for details.

### What's Next?

- [Configure environment variables](/docs/v0.3.0-Beta/configuration) for hosting providers
- [Set up admin permissions](/docs/v0.3.0-Beta/permissions) and add moderators
- [Manage admin accounts](/docs/v0.3.0-Beta/admin-manager) for staff onboarding and permission presets
- [Connect your Discord bot](/docs/v0.3.0-Beta/discord) for status embeds, log routing, and notifications
