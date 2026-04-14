# Addon Development

Build addons that extend fxPanel with custom routes, panel pages, widgets, and more.

Each addon's server code runs in an **isolated child process** — it cannot crash fxPanel and cannot access core secrets or other addons' data.

---

## Introduction

The fxPanel addon system lets you extend fxPanel's functionality without modifying core source files. Addons can add:

- **Backend API routes** — Custom HTTP endpoints proxied through fxPanel's auth layer
- **Panel pages** — Full pages accessible from the web panel via their own route
- **Panel widgets** — Components injected into existing pages (dashboard, player modal, etc.)
- **Real-time push** — WebSocket events pushed to panel clients
- **Event listeners** — React to game events like player joins/drops

### Limitations (v1)

- No hot-reload (requires a restart after install/update/removal)
- Cannot communicate with other addons directly
- Cannot access fxPanel internals or the database directly
- No auto-update

---

## Getting Started

### Directory Layout

Place your addon folder inside `addons/` at the fxPanel root:

```
fxPanel/
├── addons/
│   └── my-addon/           ← Your addon
│       ├── addon.json       ← Manifest (required)
│       ├── package.json     ← Must include "type": "module"
│       ├── server/
│       │   └── index.js     ← Server entry point
│       ├── panel/
│       │   ├── index.js     ← Panel entry (exports React components)
│       │   └── index.css    ← Styles (optional)
│       └── static/          ← Static assets (optional)
├── addon-sdk/               ← SDK (shipped with fxPanel, do not modify)
├── core/
├── panel/
└── ...
```

### Minimum Viable Addon

**1. Create the directory:**

```
addons/hello-world/
```

**2. Create `addon.json`:**

```json
{
  "id": "hello-world",
  "name": "Hello World",
  "description": "A minimal addon example",
  "version": "1.0.0",
  "author": "YourName",
  "fxpanel": {
    "minVersion": "0.1.0"
  },
  "permissions": {
    "required": ["storage"],
    "optional": []
  },
  "server": {
    "entry": "server/index.js"
  }
}
```

**3. Create `package.json`:**

```json
{
  "private": true,
  "type": "module"
}
```

**4. Create `server/index.js`:**

```js
import { createAddon } from 'addon-sdk';

const addon = createAddon();

addon.registerRoute('GET', '/hello', async (req) => {
  return { status: 200, body: { message: `Hello, ${req.admin.name}!` } };
});

addon.log.info('Hello World addon loaded');
addon.ready();
```

**5.** Restart fxPanel, then approve the addon in the **Addons** page (requires `all_permissions`).

**6. Test it:**

```
GET /addons/hello-world/api/hello
```

Returns: `{ "message": "Hello, admin!" }`

---

## Addon Manifest

The `addon.json` file is required in every addon's root directory. It is validated at boot using Zod. Invalid manifests cause the addon to be skipped with a warning.

### Full Schema

```json
{
  // ── Identity (all required) ──
  "id": "my-addon",              // Must match directory name. Lowercase a-z, 0-9, hyphens. 3-64 chars.
  "name": "My Addon",            // Display name (max 64 chars)
  "description": "What it does", // Max 256 chars
  "version": "1.0.0",            // Semver
  "author": "YourName",          // Max 64 chars
  "homepage": "https://...",     // Optional URL
  "license": "MIT",              // Optional

  // ── Compatibility ──
  "fxpanel": {
    "minVersion": "0.1.0",       // Minimum fxPanel version required
    "maxVersion": "1.0.0"        // Optional upper bound
  },

  // ── Permissions ──
  "permissions": {
    "required": ["storage"],     // Must all be granted or addon won't start
    "optional": ["ws.push"]      // Admin can choose to grant these
  },

  // ── Server entry (optional) ──
  "server": {
    "entry": "server/index.js"   // Relative to addon root
  },

  // ── Panel entry (optional) ──
  "panel": {
    "entry": "panel/index.js",
    "styles": "panel/index.css",
    "pages": [
      {
        "path": "/notes",
        "title": "Player Notes",
        "icon": "StickyNote",          // Lucide icon name
        "sidebar": true,
        "sidebarGroup": "Players",
        "permission": "players.read",
        "component": "PlayerNotesPage" // Named export from panel entry
      }
    ],
    "widgets": [
      {
        "slot": "dashboard.main",
        "component": "MyWidget",       // Named export from panel entry
        "title": "Widget Title",
        "defaultSize": "half",         // "full" | "half" | "quarter"
        "permission": "players.read"
      }
    ]
  },

  // ── NUI entry (optional) ──
  "nui": {
    "entry": "nui/index.js",
    "styles": "nui/index.css",
    "pages": [
      {
        "id": "my-page",
        "title": "My Page",
        "icon": "StickyNote",
        "component": "MyNuiPage",
        "permission": "players.read"
      }
    ]
  },

  // ── Lua resource scripts (optional) ──
  "resource": {
    "server_scripts": ["resource/sv_main.lua"],
    "client_scripts": ["resource/cl_main.lua"]
  }
}
```

### Widget Slots

Slot names use a convention-based dot-separated pattern (e.g. `dashboard.main`). You can use any slot name matching the regex `^[a-z][a-z0-9-]*(\.[a-z][a-z0-9-]*)*$` — if the panel has a matching hook call, your widget will render there.

#### Built-in Slot Points

| Slot | Location | Description |
|---|---|---|
| `dashboard.main` | Dashboard page | Main content area grid |
| `dashboard.sidebar` | Dashboard page | Right sidebar |
| `header.dropdown` | Header avatar menu | Items after Logout in user dropdown |
| `player-modal.tabs` | Player modal | Additional tabs in the player detail modal |
| `player-modal.actions` | Player modal | Extra action items in the footer dropdown |
| `server.status-cards` | Server sidebar | Status cards below server schedule |
| `settings.tab` | Settings page | Adds an entirely new settings tab |
| `settings.tab.<tabId>` | Settings page | Injects within an existing tab |
| `settings.sections` | Settings page | Widgets in the Addons management tab |

Tab IDs match the lowercase, hyphenated form of the built-in tab name (e.g. `general`, `fxserver`, `discord`).

---

## Server-Side Development

### The SDK

Every addon's server entry imports from `addon-sdk` (shipped with fxPanel). The SDK is automatically resolved via `NODE_PATH` — no `npm install` needed.

```js
import { createAddon } from 'addon-sdk';

const addon = createAddon();
```

`createAddon()` returns an addon object with these properties:

| Property | Type | Description |
|---|---|---|
| `id` | string | The addon's unique identifier (read-only) |
| `registerRoute` | function | Register HTTP route handlers |
| `storage` | AddonStorage | Scoped key-value storage |
| `ws` | AddonWebSocket | Real-time WebSocket push |
| `on` | function | Subscribe to core events |
| `log` | AddonLog | Structured logging |
| `ready` | function | Signal that the addon is initialized |

### Registering Routes

Routes are registered with `addon.registerRoute(method, path, handler)`. The path supports Express-style parameters:

```js
addon.registerRoute('GET', '/players/:license/notes', async (req) => {
  // req.method    — 'GET'
  // req.path      — '/players/abc123/notes'
  // req.params    — { license: 'abc123' }
  // req.headers   — incoming HTTP headers
  // req.body      — parsed JSON body (POST/PUT/PATCH)
  // req.admin     — { name, permissions, isMaster, hasPermission(p) }

  return {
    status: 200,
    headers: { 'X-Custom': 'hi' },   // Optional
    body: { notes: [] },
  };
});

addon.registerRoute('POST', '/players/:license/notes', async (req) => {
  const { text } = req.body;
  if (!text) return { status: 400, body: { error: 'text is required' } };

  if (!req.admin.hasPermission('players.write')) {
    return { status: 403, body: { error: 'Insufficient permissions' } };
  }

  // ... save note ...
  return { status: 200, body: { success: true } };
});
```

**How routing works:** When a request hits `GET /addons/my-addon/api/players/abc123/notes`, fxPanel authenticates the request, extracts the addon ID and remaining path, forwards everything to the addon's child process via IPC, the SDK matches the route, and the return value becomes the HTTP response.

### Signaling Ready

You **must** call `addon.ready()` after registering all routes. The core waits for this signal (default timeout: 10 seconds). If not received, the addon is marked as `failed`.

### Logging

```js
addon.log.info('Something happened');
addon.log.warn('Something suspicious');
addon.log.error('Something broke');
```

Logs are routed to fxPanel's console under the `[addon:my-addon]` prefix. Messages are truncated at 2000 characters.

### Error Handling

Unhandled exceptions and promise rejections are caught automatically. If a route handler throws, the SDK returns a `500` response with `{ "error": "Internal addon error" }`.

---

## Panel UI Development

### Entry File

Your `panel/index.js` must export named React components matching the `component` values declared in the manifest:

```js
// panel/index.js
export function MyPage() {
  return React.createElement('div', null, 'Hello from addon page!');
}

export function MyWidget() {
  return React.createElement('div', null, 'Widget content');
}
```

### Shared Dependencies

React is exposed as a global (`window.React`) by the addon loader — do **not** bundle it. If using a bundler, mark `react` and `react-dom` as externals.

### API Calls from the Panel

Panel components can call addon server routes using `fetch`. Addon API routes are at `/addons/<id>/api/<path>`. All authenticated requests must include the CSRF token header:

```js
const api = globalThis.txAddonApi;

// Get headers for authenticated requests
const headers = api.getHeaders();
// → { 'Content-Type': 'application/json', 'X-TxAdmin-CsrfToken': '...' }

// GET request
const res = await fetch('/addons/my-addon/api/data', {
  credentials: 'same-origin',
  headers: api.getHeaders(),
});

// POST request
const res = await fetch('/addons/my-addon/api/data', {
  method: 'POST',
  credentials: 'same-origin',
  headers: api.getHeaders(),
  body: JSON.stringify({ key: 'value' }),
});
```

#### txAddonApi Properties

| Property | Type | Description |
|---|---|---|
| `csrfToken` | string | Current CSRF token for the session |
| `getHeaders()` | () => object | Returns headers with Content-Type and CSRF token |
| `openPlayerModal(ref)` | (ref) => void | Opens the player modal |
| `ui` | object | Shared UI component primitives |

### Styles

If your manifest declares `panel.styles`, the CSS file is injected server-side into the HTML `<head>` as a `<link>` tag. This is useful for theme addons. Use Tailwind utility classes or scope your styles with a unique prefix to avoid conflicts.

### Building Panel Bundles

```bash
# Example with esbuild
esbuild src/panel.tsx --bundle --format=esm \
  --outfile=panel/index.js \
  --external:react --external:react-dom \
  --target=es2020
```

- Output format: **ESM**
- Externalize React (it's provided by the host)
- No filename hashing (the manifest references a fixed path)
- Target: `es2020` or later

---

## NUI (In-Game) Development

Addons can inject styles and scripts into the in-game NUI (the admin menu overlay). This works similarly to panel addon loading but targets the FiveM/RedM CEF browser.

### NUI Entry File

The NUI entry is a plain JavaScript file (no ES module exports needed). It executes in the global scope:

```js
// nui/index.js
(function () {
  var observer = new MutationObserver(function () {
    var imgs = document.querySelectorAll('img[alt="fxPanel logo"]');
    imgs.forEach(function (img) {
      img.src = txNuiAddonApi.getStaticUrl('my-addon', 'logo.svg');
    });
  });
  observer.observe(document.body, { childList: true, subtree: true });
})();
```

### txNuiAddonApi Global

| Method | Description |
|---|---|
| `getStaticUrl(addonId, filePath)` | Returns a resource-relative URL to a file in the addon's `static/` directory |
| `fetch(path, opts?)` | Makes an authenticated request to an addon API route via WebPipe |

### Differences from Panel Addons

| | Panel | NUI |
|---|---|---|
| Loading | ES module import() | `<script>` tag injection |
| Exports | widgets, pages objects | Not required (global scope) |
| React | Available via window.React | Not available (MUI-based) |
| API | window.txAddonApi | window.txNuiAddonApi |
| Files | HTTP server routes | nui://monitor/ resource protocol |

---

## Permissions

Addons request permissions in their manifest. Permissions control what the addon can do at runtime.

### Available Permissions

| Permission | Grants |
|---|---|
| `storage` | Read/write to addon's own scoped key-value store |
| `players.read` | Read player data, receive player events |
| `players.write` | Modify player data |
| `players.kick` | Kick players |
| `players.warn` | Warn players |
| `players.ban` | Ban players |
| `server.read` | Read server status, resource list, server events |
| `server.announce` | Send server-wide announcements |
| `server.command` | Execute server console commands (dangerous) |
| `database.read` | Read-only access to player/action database |
| `http.outbound` | Make outbound HTTP requests |
| `ws.push` | Push real-time data to panel clients via WebSocket |

### Required vs Optional

- **Required** — All must be granted by an admin or the addon won't start. Adding a new required permission in an update requires re-approval.
- **Optional** — The admin can choose to grant these. Check at runtime whether they were granted.

### How Approval Works

1. Place the addon in `addons/`
2. Restart fxPanel — the addon is discovered and appears as "discovered" (pending)
3. An admin with `all_permissions` navigates to the Addons page
4. Reviews the addon's requested permissions and clicks **Approve**
5. The approval is stored in `addon-config.json`
6. On next restart, the addon starts with the granted permissions

---

## Storage

Each addon gets a scoped key-value store. Data is persisted to `addon-data/<addon-id>.json` by the core — your addon never reads/writes this file directly.

### API

```js
// Get a value (returns undefined if not found)
const notes = await addon.storage.get('notes:player123');

// Set a value (JSON-serializable)
await addon.storage.set('notes:player123', [
  { text: 'Known griefer', author: 'admin1', date: '2026-04-10' }
]);

// Delete a key
await addon.storage.delete('notes:player123');

// List keys by prefix
const keys = await addon.storage.list('notes:');
// → ['notes:player123', 'notes:player456']
```

### Limits

- **10 MB** max storage per addon (configurable by the server admin)
- Writes are debounced — flushed to disk every 5 seconds or on shutdown
- Keys are strings, values must be JSON-serializable
- Storage operations timeout after 5 seconds

---

## Real-Time WebSocket Push

Addons with the `ws.push` permission can push real-time events to panel clients.

### Server Side

```js
// Push an event to all subscribed panel clients
addon.ws.push('notes:updated', { license: 'abc123', count: 5 });

// React to panel clients subscribing/unsubscribing
addon.ws.onSubscribe((sessionId) => {
  addon.log.info(`Client subscribed: ${sessionId}`);
});

addon.ws.onUnsubscribe((sessionId) => {
  addon.log.info(`Client left: ${sessionId}`);
});
```

### Panel Side

Panel clients join the `addon:<addonId>` Socket.io room. The panel addon loader handles this automatically when addon components mount. Events are emitted as `addon:<addonId>:<eventName>` on the socket.

---

## Events

Addons can listen for game events broadcast by fxPanel core.

```js
addon.on('playerJoining', (data) => {
  addon.log.info(`Player joining: ${data.displayName} (${data.license})`);
});

addon.on('playerDropped', (data) => {
  addon.log.info(`Player dropped: netid=${data.netid}, reason=${data.reason}`);
});
```

### Available Events

**`playerJoining`**
Fired when a player connects.
`{ netid, displayName, license, ids }`

**`playerDropped`**
Fired when a player disconnects.
`{ netid, reason }`

**`playerBanned`**
Fired when a player is banned.
`{ author, reason, actionId, expiration, ... }`

**`playerWarned`**
Fired when a player is warned.
`{ author, reason, actionId, targetNetId, ... }`

**`playerKicked`**
Fired when a player is kicked.
`{ target, author, reason, dropMessage }`

---

## Player Tags

Addons with the `players.write` permission can programmatically add or remove custom tags on connected players using the `addon.players` API. Tags must be pre-defined in the server's `customTags` configuration.

### API

```js
// Add a tag to a connected player
await addon.players.addTag(netid, 'vip');
// → resolves true on success

// Remove a tag from a connected player
await addon.players.removeTag(netid, 'vip');
// → resolves true on success
```

### Parameters

| Parameter | Type | Description |
|---|---|---|
| `netid` | number | The player's server network ID (must be connected and registered) |
| `tagId` | string | The ID of a custom tag defined in the server's `customTags` config |

### Requirements

- The addon must have the `players.write` permission granted.
- The `tagId` must match a tag defined in the server's `gameFeatures.customTags` configuration.
- The target player must be currently connected and registered (has a valid license).

### Error Handling

The promise rejects with a descriptive error string if:

- The `players.write` permission is not granted
- Arguments are invalid (netid not a number, tagId not a string)
- The tag ID is not found in the server's custom tags configuration
- The player is not found or not registered

### Example

```js
addon.on('playerJoining', async (data) => {
  // Automatically tag VIP players when they join
  const vipLicenses = await addon.storage.get('vip-list') || [];
  if (vipLicenses.includes(data.license)) {
    try {
      await addon.players.addTag(data.netid, 'vip');
      addon.log.info(`Tagged player #${data.netid} as VIP`);
    } catch (err) {
      addon.log.error(`Failed to tag player: ${err}`);
    }
  }
});
```

---

## Addon Admin Management

Admins with `all_permissions` can access the Addons page from the global menu. This page shows all discovered addons with their current state, metadata, and requested permissions.

### Addon States

| State | Meaning |
|---|---|
| `discovered` | Found on disk, pending admin approval |
| `approved` | Permissions granted, will start on next boot |
| `starting` | Process is spawning |
| `running` | Active and serving requests |
| `stopping` | Graceful shutdown in progress |
| `stopped` | Explicitly disabled by admin |
| `failed` | Could not start (entry file missing, timeout, error) |
| `crashed` | Process exited unexpectedly at runtime |
| `invalid` | Manifest validation failed |

---

## File Structure Reference

### What an Addon Provides

```
addons/my-addon/
├── addon.json          ← Required manifest
├── package.json        ← Must have "type": "module"
├── server/
│   └── index.js        ← Server entry (runs in child process)
├── panel/
│   ├── index.js        ← Panel bundle (exports React components)
│   └── index.css       ← Optional styles
├── nui/
│   ├── index.js        ← NUI bundle
│   └── index.css       ← Optional styles
├── resource/
│   ├── sv_main.lua     ← Server-side Lua
│   └── cl_main.lua     ← Client-side Lua
└── static/
    └── icon.png        ← Served at /addons/my-addon/static/icon.png
```

All directories except `addon.json` and `package.json` are optional — include only the layers you need.

### What fxPanel Generates

```
addon-data/                ← Managed by core, per-profile
├── my-addon.json          ← Storage data for my-addon
└── another-addon.json

addon-config.json          ← Per-profile approval + settings
```

---

## Security Notes

### Process Isolation

Your addon's server code runs in a separate Node.js process with a minimal environment. It has **no access** to:

- fxPanel's database connections
- Admin sessions or tokens
- Environment variables (except `NODE_ENV` and `ADDON_ID`)
- Other addons' processes or storage

### HTTP Safety

- All addon API requests pass through fxPanel's auth middleware
- Response headers are sanitized: `Set-Cookie` and `Content-Security-Policy` are stripped
- Addon routes are namespaced under `/addons/<id>/api/`

### What to Avoid

- Do not shell out / spawn child processes from your addon
- Do not attempt to read files outside your addon directory
- Do not store sensitive data (tokens, passwords) in addon storage — it's a plain JSON file on disk

---

## Troubleshooting

### Addon isn't discovered

- Verify the directory name matches the `id` field in `addon.json`
- Check the console for Zod validation errors on the manifest
- Ensure `addon.json` exists in the addon root (not in a subdirectory)

### Addon stays in "discovered" state

- It needs to be approved by an admin with `all_permissions`
- After approval, fxPanel must be restarted

### Addon fails to start

- Check the console for `[addon:my-addon]` error messages
- Missing `"type": "module"` in `package.json`
- Syntax error in `server/index.js`
- `addon.ready()` never called (10-second timeout)
- SDK import must be `from 'addon-sdk'`

### API routes return 503

- The addon process is not running (check state in Addons page)
- The addon crashed — check console logs for the error

### Storage operations fail

- Ensure `storage` is listed in `permissions.required` and was approved
- Storage operations timeout after 5 seconds

### Panel component doesn't render

- Verify your entry file exports a function matching the `component` name in the manifest
- Check the browser console for import or render errors
- Ensure React is not bundled in your panel entry (use externals)

### WebSocket push not working

- Ensure `ws.push` is in your granted permissions
- Panel clients must be subscribed to the addon's room (automatic for addon components)

---

## Example: discord-logger Addon

A complete working example is shipped at `addons/discord-logger/`. It demonstrates:

- Manifest with server entry + panel widget
- CRUD routes for webhook config and log retrieval
- Scoped storage usage
- WebSocket push on new log entries
- Panel widget injected into the Discord settings tab
- Authenticated API calls from panel using `txAddonApi.getHeaders()`
- Admin permission checking in routes
- Event listeners for `playerBanned`, `playerWarned`, and `playerKicked`
- Discord webhook integration with embed formatting
