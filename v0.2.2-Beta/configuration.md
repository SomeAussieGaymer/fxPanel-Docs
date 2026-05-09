# Configuration

Environment variables and settings to customize your fxPanel instance.

---

## Environment Variables

fxPanel is configured primarily through environment variables, making it easy for hosting providers (GSPs) to pre-configure instances. Set these before starting the FXServer process.

> **Note:** Environment variables are typically set in an `.env` file located in fxPanel's main directory, or passed directly through your server's launch configuration. All variables use the `TXHOST_` prefix. Do not wrap values in quotes — fxPanel will reject quoted values and show an error.

| Variable | Description | Default |
|----------|-------------|---------|
| `TXHOST_DATA_PATH` | Path to the txData folder | OS-dependent |
| `TXHOST_GAME_NAME` | Restrict to `fivem` or `redm` | auto-detect |
| `TXHOST_MAX_SLOTS` | Enforce maximum player count | — |
| `TXHOST_QUIET_MODE` | Hide FXServer output in stdout | false |
| `TXHOST_API_TOKEN` | Token for the `/host/status` endpoint | — |

---

## Networking

Control how fxPanel binds to network interfaces and which ports it uses.

| Variable | Description | Default |
|----------|-------------|---------|
| `TXHOST_TXA_URL` | Public URL for fxPanel | — |
| `TXHOST_TXA_PORT` | fxPanel web interface port | 40120 |
| `TXHOST_FXS_PORT` | Force a specific FXServer port | — |
| `TXHOST_INTERFACE` | Bind interface address | 0.0.0.0 |

---

## Provider Customization

Hosting providers can brand the fxPanel interface with their name and logo.

| Variable | Description |
|----------|-------------|
| `TXHOST_PROVIDER_NAME` | Hosting provider name displayed in the panel |
| `TXHOST_PROVIDER_LOGO` | URL to provider logo (max 224×96 px) |

---

## Default Values

Pre-fill setup fields so server owners don't have to enter them manually. These are used as defaults during the initial setup wizard and deployer.

| Variable | Description |
|----------|-------------|
| `TXHOST_DEFAULT_DB*` | Database credentials (host, port, username, password, name) |
| `TXHOST_DEFAULT_CFXKEY` | Default CFX license key |
| `TXHOST_DEFAULT_ACCOUNT` | Default admin account for auto-setup |

> **Security Note:** Default credentials (especially database passwords and CFX keys) should only be set in secure hosting environments. They are visible to anyone with access to environment variables on the machine.
