# Getting Started

Install fxPanel and get your first server up and running.

---

## Introduction

fxPanel is a modern, full-featured web panel and in-game menu for managing FiveM and RedM servers. It provides real-time monitoring, player management, analytics, a comprehensive permission system, Discord integration, and an automated deployment system — all from a single interface.

fxPanel replaces the need for external databases by default, using a self-contained player database with in-memory indexing. It supports 30+ languages, custom theming, and a recipe-based deployer for setting up new servers in under 60 seconds.

---

## Installation

fxPanel replaces the default txAdmin monitor that ships with FXServer. Installation steps differ slightly by platform.

### Windows

1. **Install FXServer** — Download and install any FXServer artifact from the [official Cfx.re builds page](https://runtime.fivem.net/artifacts/fivem/build_server_windows/master/).

2. **Delete the default monitor** — Navigate to `citizen\system_resources` inside the folder where you installed the FXServer artifact, and delete the `monitor` folder.

3. **Download fxPanel** — Go to the [fxPanel GitHub releases page](https://github.com/AquaPointer/fxPanel/releases) and download the latest release.

4. **Install fxPanel** — Drag and drop the downloaded fxPanel folder into `citizen\system_resources` (the same location you deleted the monitor folder from).

5. **Start FXServer** — Run `FXServer.exe`. fxPanel will automatically initialize with a profile in the `txData` folder.

6. Open your browser and navigate to `http://localhost:40120` (default port).

> ⛔ **Important:** Make sure you fully delete the existing `monitor` folder before adding fxPanel. Running both at the same time will cause conflicts.

> **Note:** The default port is `40120`. You can change it via the `TXHOST_TXA_PORT` environment variable. See [Configuration](../configuration) for details.

### Linux

1. **Install FXServer** — Download the Linux FXServer artifact and extract it to your desired location.

2. **Delete the default monitor** — Navigate to `citizen/system_resources` inside the FXServer directory and delete the `monitor` folder:
   ```bash
   rm -rf /path/to/fxserver/citizen/system_resources/monitor
   ```

3. **Download fxPanel** — Download the latest release from the [fxPanel GitHub releases page](https://github.com/AquaPointer/fxPanel/releases).

4. **Install fxPanel** — Extract the fxPanel folder into `citizen/system_resources`:
   ```bash
   cp -r fxPanel-v1.0.0/monitor /path/to/fxserver/citizen/system_resources/
   ```

5. **Start FXServer** — Run the server start script. fxPanel will automatically initialize with a profile in the `txData` folder:
   ```bash
   ./run.sh +exec server.cfg
   ```

6. Open your browser and navigate to `http://your-server-ip:40120`.

> **Note:** Ensure the `monitor` directory has the correct ownership and permissions for the user running FXServer.

### Existing Installations

The same steps above can be followed to install fxPanel on servers that already have txAdmin or a previous version of fxPanel installed. Simply delete the existing `monitor` folder and replace it with the latest fxPanel release.

> ⚠️ **Warning — Potential Data Loss:** Installing fxPanel on an existing server may result in the loss or corruption of your `txData` folder, which contains your server profiles, player database, admin accounts, and configuration. We strongly recommend making a full backup of your `txData` folder before proceeding. The fxPanel team is not responsible for any data loss that may occur during this process.

---

## Quick Start

After installation, follow these steps to get your server running:

1. **Create a Master Account** — On first launch, you'll be prompted to create a master admin account with full permissions.

2. **Deploy a Server** — Use the built-in Deployer with a recipe to set up a server automatically, or point fxPanel at an existing server data folder.

3. **Configure Settings** — Visit the Settings page to configure your server name, CFX license key, database connection, Discord bot, and more.

4. **Start the Server** — Click the Start button on the dashboard. fxPanel will manage the FXServer process with automatic crash/hang recovery.

5. **Access In-Game** — Join your server and type `/tx` to open the admin menu. See [In-Game Menu](../in-game-menu) for details.

### What's Next?

- [Configure environment variables](../configuration) for hosting providers
- [Set up admin permissions](../permissions) and add moderators
- [Connect your Discord bot](../discord) for status embeds and notifications
