# Artifact Updater

The Artifact Updater manages FXServer runtime builds from the panel. It is available at `/system/artifacts` and requires `all_permissions`.

Use this page only for planned maintenance. Applying an artifact update stops the game server, swaps runtime files, and restarts the fxPanel process when possible.

## Architecture

The panel page is implemented in `panel/src/pages/FxUpdater/FxUpdaterPage.tsx`. The backend updater is implemented by `core/modules/FxUpdater/index.ts` and exposed through three authenticated routes.

| Route | Method | Purpose |
| ----- | ------ | ------- |
| `/fxserver/artifacts` | `GET` | Returns installed build, version tag, available artifact tiers, and updater status. |
| `/fxserver/artifacts/download` | `POST` | Validates the request, logs the action, and starts a background download from an approved URL. |
| `/fxserver/artifacts/apply` | `POST` | Applies an already extracted update if the updater is in `extracted` state. |

The artifact list is fetched from the FiveM changelog API for the current host platform:

- Windows: `win32/server`
- Linux: `linux/server`

Returned tiers are `latest`, `recommended`, `optional`, and `critical`.

## Update Flow

Updater status is one of:

| Phase | Meaning |
| ----- | ------- |
| `idle` | No update is running. |
| `downloading` | Archive download is active and includes a percentage. |
| `extracting` | Archive is being unpacked into staging. |
| `extracted` | Staging is ready to apply. |
| `applying` | The updater is stopping FXServer and swapping files. |
| `error` | The updater failed and includes a message. |

As currently implemented, the backend `download()` flow continues into `apply()` after extraction succeeds. Treat the **Download** action as the start of the full disruptive update unless the implementation is changed.

## Download Sources

Built-in artifact tiers use download URLs returned by the FiveM changelog API. Custom URLs must be HTTPS and hosted on:

```text
runtime.fivem.net
```

The panel validates the same hostname before sending the request, and the backend rejects any other hostname.

Example custom URL shape:

```text
https://runtime.fivem.net/artifacts/fivem/build_server_windows/master/<build>-<hash>/server.zip
```

Linux artifacts use the Linux artifact directory and `fx.tar.xz` archive format.

## Platform Behavior

The updater stages files beside the current artifact root:

- Temporary download directory: `fxserver_update_temp`
- Extracted staging directory: `fxserver_update_staging`
- Failure marker: `fxserver_update_failure.txt`

On Windows, `txEnv.fxsPath` is treated as the artifact root. The updater writes a detached `fxs_update_swap.bat` script, waits for the fxPanel process to stop, preserves `citizen/` where possible, swaps or copies staged files into place, then attempts to restart using the original command line.

On Linux, `txEnv.fxsPath` points at `alpine/opt/cfx-server/`, so the artifact root is resolved two directories above that path. The updater accepts archives that extract directly to the artifact root or through an `alpine/` wrapper, validates `opt/cfx-server/FXServer` and `ld-musl-x86_64.so.1`, then uses a detached shell script to swap the staged artifact.

## Constraints

- Only admins with `all_permissions` can list, download, or apply artifacts.
- The updater's `download()` method refuses to start while `downloading` or `applying`. The route starts downloads in the background, so those internal busy failures are not returned synchronously to the original HTTP response.
- Apply is rejected unless status is `extracted`.
- Downloads use a 60-minute request timeout and stream progress.
- ZIP extraction validates paths to prevent zip-slip writes outside staging.
- Download or apply failures write the failure marker so the next startup can surface persisted updater errors.

## Operator Checklist

1. Warn players and schedule downtime before clicking **Download**.
2. Confirm the current build and selected tier on `/system/artifacts`.
3. Use a built-in tier when possible; use a custom URL only from `runtime.fivem.net`.
4. Watch the updater status until the process restarts or reports an error.
5. If restart cannot be detected automatically, restart FXServer manually and check for `fxserver_update_failure.txt` beside the artifact root.
