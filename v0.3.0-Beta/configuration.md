# Configuration

Environment variables and settings to customize your fxPanel instance.

---

## Environment Variables

fxPanel host configuration is read from process environment variables with the `TXHOST_` prefix. Set these in the FXServer launch environment before starting the process.

> **Note:** The production runtime reads `process.env`; it does not load a `.env` file by itself. The repository's dev scripts load the root `.env` for `TXDEV_*` workflows, but `TXHOST_*` values for a deployed server should come from your shell, service manager, container, or hosting panel. Do not wrap values in quotes; fxPanel rejects quoted `TXHOST_*` values.

| Variable | Description | Default |
|----------|-------------|---------|
| `TXHOST_DATA_PATH` | Absolute path to the `txData` folder | Derived from FXServer path |
| `TXHOST_GAME_NAME` | Restrict game selection to `fivem` or `redm` | Auto-detect |
| `TXHOST_MAX_SLOTS` | Enforce a positive integer maximum player count | Unset |
| `TXHOST_QUIET_MODE` | Set to `true` to hide FXServer output in stdout | `false` |
| `TXHOST_API_TOKEN` | Token for `GET /host/status`; use `disabled` to disable the endpoint | Unset |

`TXHOST_API_TOKEN` must be 16-48 characters and may contain letters, numbers, underscores, and dashes.

---

## Networking

Control how fxPanel binds to network interfaces and which ports it uses.

| Variable | Description | Default |
|----------|-------------|---------|
| `TXHOST_TXA_URL` | Public URL for fxPanel | Unset |
| `TXHOST_TXA_PORT` | fxPanel web interface port; cannot be `30120` | `40120` |
| `TXHOST_FXS_PORT` | Force a specific positive FXServer port; cannot be `40120`-`40150` | Unset |
| `TXHOST_INTERFACE` | Bind interface IPv4 address | Default bind behavior |

---

## Provider Customization

Hosting providers can brand the fxPanel interface with their name and logo.

| Variable | Description |
|----------|-------------|
| `TXHOST_PROVIDER_NAME` | Hosting provider name displayed in the panel |
| `TXHOST_PROVIDER_LOGO` | URL to provider logo |

Provider names must be 2-16 characters, start and end with a letter or number, and may contain letters, numbers, spaces, underscores, periods, and hyphens. Consecutive special characters are rejected.

---

## Default Values

Pre-fill setup fields so server owners don't have to enter them manually. These are used as defaults during the initial setup wizard and deployer.

| Variable | Description |
|----------|-------------|
| `TXHOST_DEFAULT_DBHOST` | Default database host |
| `TXHOST_DEFAULT_DBPORT` | Default database port |
| `TXHOST_DEFAULT_DBUSER` | Default database username |
| `TXHOST_DEFAULT_DBPASS` | Default database password |
| `TXHOST_DEFAULT_DBNAME` | Default database name |
| `TXHOST_DEFAULT_CFXKEY` | Default CFX license key |
| `TXHOST_DEFAULT_ACCOUNT` | Default admin account for auto-setup |

> **Security Note:** Default credentials (especially database passwords and CFX keys) should only be set in secure hosting environments. They are visible to anyone with access to environment variables on the machine.

`TXHOST_DEFAULT_ACCOUNT` must be `username:fivemId` or `username:fivemId:bcrypt`. The username must pass the same FiveM username validation used for admin accounts, and the optional password must be a bcrypt hash.
