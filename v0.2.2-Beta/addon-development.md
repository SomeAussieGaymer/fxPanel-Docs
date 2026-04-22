# Addon Development

Build addons that extend fxPanel with custom routes, panel pages, widgets, and more.

> Each addon's server code runs in an **isolated child process** — it cannot crash fxPanel and cannot access core secrets or other addons' data.

---

## Introduction

The fxPanel addon system lets you extend fxPanel's functionality without modifying core source files. Addons can add:

- **Backend API routes** — Custom HTTP endpoints proxied through fxPanel's auth layer
- **Public routes** — Unauthenticated HTTP endpoints served on a dedicated port (websites, APIs, SPAs)
- **Panel pages** — Full pages accessible from the web panel via their own route
- **Panel widgets** — Components injected into existing pages (dashboard, player modal, etc.)
- **Real-time push** — WebSocket events pushed to panel clients
- **Event listeners** — React to game events like player joins/drops

### Current Limitations

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
    "minVersion": "0.2.1-Beta"
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
  // — Identity (all required) —
  "id": "my-addon",              // Must match directory name. Lowercase a-z, 0-9, hyphens. 3-64 chars.
  "name": "My Addon",            // Display name (max 64 chars)
  "description": "What it does", // Max 256 chars
  "version": "1.0.0",            // Semver
  "author": "YourName",          // Max 64 chars
  "homepage": "https://...",     // Optional URL
  "license": "MIT",              // Optional

  // — Compatibility —
  "fxpanel": {
    "minVersion": "0.2.1-Beta",  // Minimum fxPanel version required
    "maxVersion": "1.0.0"        // Optional upper bound
  },

  // — Permissions —
  "permissions": {
    "required": ["storage"],     // Must all be granted or addon won't start
    "optional": ["ws.push"]      // Admin can choose to grant these
  },

  // — Dependencies (optional) —
  "dependencies": [
    "other-addon"                // Addon IDs that must be running before this addon starts
  ],

  // — Public Routes (optional) —
  "publicRoutes": true,          // Enable unauthenticated public route registration
  "publicServer": {
    "defaultPort": 8080          // Default dedicated port (admin can override)
  },

  // — Server entry (optional) —
  "server": {
    "entry": "server/index.js"   // Relative to addon root
  },

  // — Panel entry (optional) —
  "panel": {
    "entry": "panel/index.js",
    "styles": "panel/index.css",
    "pages": [
      {
        "path": "/notes",
        "title": "Player Notes",
        "icon": "StickyNote",          // Lucide icon name
        "sidebar": true,               // Show in sidebar + top nav (default: false = top nav only)
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

  // — NUI entry (optional) —
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

  // — Custom Admin Permissions (optional) —
  "adminPermissions": [
    {
      "id": "my-addon.manage",
      "label": "Manage My Addon",
      "description": "Allows managing the addon's configuration"
    }
  ],

  // — Lua resource scripts (optional) —
  "resource": {
    "server_scripts": ["resource/sv_main.lua"],
    "client_scripts": ["resource/cl_main.lua"]
  }
}
```

### Page Visibility

The `sidebar` property controls where your addon page appears in the panel navigation:

| `sidebar` value | Sidebar (left nav) | Top nav bar | Mobile menu |
|-----------------|--------------------|--------------------|-------------|
| `true` | Yes | Yes | Yes |
| `false` (default) | No | Yes | Yes |

All addon pages always appear in the top navigation bar (either as a direct link if there's only one, or under an "Addons" dropdown if there are multiple). Setting `sidebar: true` additionally pins the page to the left sidebar for quick access.

### Widget Slots

Slot names use a convention-based dot-separated pattern (e.g. `dashboard.main`). You can use any slot name matching the regex `^[a-z][a-z0-9-]*(\.[a-z][a-z0-9-]*)*$` — if the panel has a matching hook call, your widget will render there.

#### Built-in Slot Points

| Slot | Location | Description |
|------|----------|-------------|
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
|----------|------|-------------|
| `id` | `string` | The addon's unique identifier (read-only) |
| `permissions` | `string[]` | The permissions granted to this addon (read-only) |
| `registerRoute` | `function` | Register HTTP route handlers |
| `registerPublicRoute` | `function` | Register unauthenticated public route handlers |
| `storage` | `AddonStorage` | Scoped key-value storage |
| `ws` | `AddonWebSocket` | Real-time WebSocket push |
| `on` | `function` | Subscribe to core events |
| `off` | `function` | Unsubscribe from core events |
| `log` | `AddonLog` | Structured logging |
| `ready` | `function` | Signal that the addon is initialized |

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

### Wildcard Routes

Routes support a `*` wildcard segment for catch-all matching. This is useful for SPA routing or serving file trees:

```js
addon.registerPublicRoute('GET', '/assets/*', async (req) => {
  const filePath = req.params['*']; // e.g. "css/main.css" or "js/app.js"
  // ... serve the file ...
});

// Catch-all for SPA routing
addon.registerPublicRoute('GET', '/*', async (req) => {
  return { status: 200, headers: { 'Content-Type': 'text/html' }, body: indexHtml };
});
```

The wildcard matches all remaining path segments and is available as `req.params['*']`.

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

## Public Routes

> **⚠️ Security Notice:** Public routes are unauthenticated — anyone with network access to the port can reach them. Do not expose sensitive data or admin functionality through public routes. Always validate and sanitize input.

Public routes let addons serve content without requiring admin authentication. This is ideal for community websites, application forms, server rules pages, REST APIs, and single-page applications.

### Manifest Setup

Enable public routes in your `addon.json`:

```json
{
  "publicRoutes": true,
  "publicServer": {
    "defaultPort": 8080
  }
}
```

### Registering Public Routes

Use `addon.registerPublicRoute(method, path, handler)` — same signature as `registerRoute` but the `req.admin` field is always `null`.

```js
// Serve a home page
addon.registerPublicRoute('GET', '/', async (req) => {
  return {
    status: 200,
    headers: { 'Content-Type': 'text/html' },
    body: '<html><body><h1>Welcome to our server!</h1></body></html>',
  };
});

// Dynamic page route
addon.registerPublicRoute('GET', '/page/:slug', async (req) => {
  const { slug } = req.params;
  const content = await addon.storage.get(`page:${slug}`);
  if (!content) return { status: 404, body: { error: 'Page not found' } };
  return {
    status: 200,
    headers: { 'Content-Type': 'text/html' },
    body: content,
  };
});

// REST API endpoint
addon.registerPublicRoute('GET', '/api/status', async (req) => {
  return {
    status: 200,
    body: { online: true, players: 64, maxPlayers: 128 },
  };
});
```

### What Can You Serve?

| Content Type | Description |
|-------------|-------------|
| Server-rendered HTML | Build HTML strings in your handler and return with `Content-Type: text/html` |
| Single-Page Applications | Serve a pre-built SPA (React, Vue, etc.) with static file serving and a catch-all route |
| REST APIs | Return JSON from public routes for external integrations |
| Static files | Serve CSS, JS, images, fonts by reading files from your addon directory |
| Binary content | Serve PDFs, downloads, etc. using appropriate Content-Type headers |

#### Serving a Pre-Built SPA

```js
import { readFileSync } from 'node:fs';
import { join } from 'node:path';

const DIST_DIR = join(import.meta.dirname, '..', 'dist');
const indexHtml = readFileSync(join(DIST_DIR, 'index.html'), 'utf-8');

const MIME_TYPES = {
  '.js': 'application/javascript',
  '.css': 'text/css',
  '.svg': 'image/svg+xml',
  '.png': 'image/png',
  '.jpg': 'image/jpeg',
  '.woff2': 'font/woff2',
  '.json': 'application/json',
};

// Serve hashed assets with immutable caching
addon.registerPublicRoute('GET', '/assets/:file', async (req) => {
  const filePath = join(DIST_DIR, 'assets', req.params.file);

  // Prevent directory traversal
  if (!filePath.startsWith(join(DIST_DIR, 'assets'))) {
    return { status: 403, body: 'Forbidden' };
  }

  const ext = '.' + req.params.file.split('.').pop();
  const mime = MIME_TYPES[ext] || 'application/octet-stream';

  try {
    const content = readFileSync(filePath, ext === '.js' || ext === '.css' || ext === '.svg' ? 'utf-8' : null);
    return {
      status: 200,
      headers: {
        'Content-Type': mime,
        'Cache-Control': 'public, max-age=31536000, immutable',
      },
      body: content,
    };
  } catch {
    return { status: 404, body: 'Not found' };
  }
});

// SPA catch-all — serve index.html for all page routes
addon.registerPublicRoute('GET', '/', async () => {
  return { status: 200, headers: { 'Content-Type': 'text/html' }, body: indexHtml };
});

addon.registerPublicRoute('GET', '/:page', async () => {
  return { status: 200, headers: { 'Content-Type': 'text/html' }, body: indexHtml };
});
```

> **Note:** The SDK's route matcher supports simple `:param` segment matching and `*` wildcard catch-all. For SPA routing, use a wildcard route like `/*` to match all paths.

### Public Request Object

| Property | Type | Description |
|----------|------|-------------|
| `method` | `string` | HTTP method (`GET`, `POST`, etc.) |
| `path` | `string` | The matched path |
| `params` | `object` | Route parameters (e.g. `{ slug: 'about' }`) |
| `headers` | `object` | Incoming HTTP headers |
| `body` | `any` | Parsed request body (POST/PUT/PATCH) |
| `admin` | `null` | Always `null` for public routes (no authentication) |

### How Public Routes Are Accessed

| Method | URL Pattern | Description |
|--------|------------|-------------|
| Dedicated port | `http://host:<port>/` | When a public port is configured, routes are served directly |
| Fallback | `/site/<addonId>/` | If no dedicated port is configured, routes are available under this prefix on the main fxPanel port |

### Port Configuration

The public routes port is determined in this order:

1. **Admin override** — If the server admin configures a specific port in addon settings, that takes priority
2. **Manifest default** — The `publicServer.defaultPort` value from your `addon.json`
3. **Disabled** — If neither is set, the dedicated port is not started and only the `/site/<addonId>/` fallback is available

### Rate Limiting

Public routes are rate-limited to **600 requests per minute per IP address**. This protects against abuse without affecting normal usage categories like page loads and API polling.

### Security Considerations

- **Set-Cookie stripping** — Response `Set-Cookie` headers are stripped from public route responses to prevent session confusion with fxPanel's admin auth
- **XSS prevention** — If serving user-generated content, always sanitize HTML output
- **Input validation** — Treat all public route input as untrusted. Validate and sanitize `req.body`, `req.params`, and `req.headers`
- **No admin context** — `req.admin` is always `null`. Never expose admin-only data through public routes

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
|----------|------|-------------|
| `csrfToken` | `string` | Current CSRF token for the session |
| `getHeaders()` | `() => object` | Returns headers with Content-Type and CSRF token |
| `openPlayerModal(ref)` | `(ref) => void` | Opens the player modal |
| `ui` | `object` | Shared UI component primitives |

### Styles

If your manifest declares `panel.styles`, the CSS file is injected server-side into the HTML `<head>` as a `<link>` tag for authenticated users. This is useful for theme addons. Use Tailwind utility classes or scope your styles with a unique prefix to avoid conflicts.

#### Theme Addon Compatibility

Theme addons often need to style the **login page** and other unauthenticated views where addon JavaScript doesn't run. fxPanel provides a server-side compatibility layer for this:

**How it works:** If a running addon has a `static/theme.json` file with `"enabled": true` and a `panel` object containing CSS custom properties, fxPanel will:

1. **Inline the addon's panel CSS** - The `panel.styles` CSS file is embedded as an inline `<style>` tag in the HTML `<head>` for all visitors (not just authenticated users). This lets theme selectors (e.g. `html[data-addon-themer-enabled='true'] { ... }`) apply everywhere.
2. **Inject CSS custom properties** - The `panel` object from `theme.json` is rendered as CSS declarations inside a scoped `<style>` tag.
3. **Set data attributes on `<html>`** - The `data-addon-themer-enabled="true"` attribute is added to the `<html>` tag server-side, activating any CSS gated behind that selector.
4. **Embed the panel logo** - If `branding.panelLogo` is set in `theme.json`, the referenced image from `static/` is base64-encoded and injected as `window.txConsts.addonThemeLogo`. All logo components in the panel (login page, sidebar, mobile menu) use this value when present.

**`theme.json` format:**

```json
{
  "enabled": true,
  "branding": {
    "panelLogo": "my-logo.png",
    "nuiLogo": ""
  },
  "panel": {
    "--background": "221 40% 10%",
    "--foreground": "210 40% 96%",
    "--accent": "192 95% 50%"
  },
  "nui": {
    "--background": "#0c0e16",
    "--foreground": "#e5e7eb"
  }
}
```

The `panel` object keys must be valid CSS custom property names (starting with `--`). Values are HSL triplets (without the `hsl()` wrapper) matching the format used by the panel's CSS variable system.

> **Note:** This compatibility layer reads `theme.json` synchronously at HTML generation time. Changes to `theme.json` take effect on the next page load. The addon's JavaScript still runs for authenticated users and can override or extend the server-injected values at runtime.

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
|--------|-------------|
| `getStaticUrl(addonId, filePath)` | Returns a resource-relative URL to a file in the addon's `static/` directory |
| `fetch(path, opts?)` | Makes an authenticated request to an addon API route via WebPipe |

### Differences from Panel Addons

| | Panel | NUI |
|-|-------|-----|
| Loading | ES module `import()` | `<script>` tag injection |
| Exports | widgets, pages objects | Not required (global scope) |
| React | Available via `window.React` | Not available (MUI-based) |
| API | `window.txAddonApi` | `window.txNuiAddonApi` |
| Files | HTTP server routes | `nui://monitor/` resource protocol |
| Theme compat | Server-side CSS + data attrs via `theme.json` | CSS/JS loaded on first menu open |

### NUI Static Asset Resolution

When the addon manager starts addons with NUI layers, it copies the addon's `nui/` and `static/` directories into the `monitor` resource directory. This makes files accessible via the `nui://` protocol:

- `nui/index.js` -> `nui://monitor/nui/addons/<addonId>/index.js`
- `static/logo.png` -> `nui://monitor/addons/<addonId>/static/logo.png`

The `txNuiAddonApi.getStaticUrl(addonId, filePath)` helper constructs these URLs automatically. This copy happens on boot and on hot-reload.

---

## Permissions

Addons request permissions in their manifest. Permissions control what the addon can do at runtime.

### Available Permissions

| Permission | Grants |
|------------|--------|
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

// Get a value with a default fallback
const config = await addon.storage.getOr('config', { theme: 'dark', lang: 'en' });

// Check if a key exists
const exists = await addon.storage.has('notes:player123');

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

### Unsubscribing from Events

Use `addon.off()` to remove event handlers:

```js
function onJoin(data) {
  addon.log.info(`Player joining: ${data.displayName}`);
}

addon.on('playerJoining', onJoin);

// Later, remove the specific handler
addon.off('playerJoining', onJoin);

// Or remove all handlers for an event
addon.off('playerJoining');
```

### Available Events

**`playerJoining`**
Fired when a player connects.
```
{ netid, displayName, license, ids }
```

**`playerDropped`**
Fired when a player disconnects.
```
{ netid, reason }
```

**`playerBanned`**
Fired when a player is banned.
```
{ author, reason, actionId, expiration, ... }
```

**`playerWarned`**
Fired when a player is warned.
```
{ author, reason, actionId, targetNetId, ... }
```

**`playerKicked`**
Fired when a player is kicked.
```
{ target, author, reason, dropMessage }
```

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
|-----------|------|-------------|
| `netid` | `number` | The player's server network ID (must be connected and registered) |
| `tagId` | `string` | The ID of a custom tag defined in the server's `customTags` config |

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
|-------|---------|
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

## Hot-Reload

Addons support hot-reload during development. When an addon's server entry file or any of its dependencies change on disk, fxPanel can automatically restart the addon process without restarting the entire server.

### How It Works

1. fxPanel watches the addon's directory for file changes
2. When a change is detected in a server file, the addon's child process is gracefully stopped
3. A new child process is spawned with the updated code
4. The addon re-registers its routes and calls `addon.ready()`
5. The reload is logged in the console and visible in the Addons page

### Manual Reload

Admins can also trigger a manual reload from the Addons page by clicking the **Reload** button on a running addon. This performs the same graceful restart cycle.

### Panel & NUI Changes

Panel and NUI bundles are not hot-reloaded automatically. After rebuilding your panel/NUI bundles:

- **Panel**: Refresh the browser to load the updated bundle
- **NUI**: Reconnect to the server or use the `txAdmin:refreshNui` client event

### Permission Re-Approval

If an addon update changes its `permissions.required` array, the addon will be moved back to the `discovered` state and require re-approval by an admin with `all_permissions`.

---

## Dependencies

Addons can declare dependencies on other addons. This ensures that required addons are present, approved, and started in the correct order.

### Declaring Dependencies

Add a `dependencies` array to your `addon.json`:

```json
{
  "dependencies": ["data-provider", "shared-utils"]
}
```

Values are addon IDs that must be running before this addon starts.

### Behavior

- **Boot order** — Dependent addons start after their dependencies are running (topological sort)
- **Missing dependency** — If a required addon is not found or not approved, the dependent addon is marked as `failed` with a descriptive message
- **Circular or unresolvable dependency chains** — Not handled by a separate hard-stop pass. Addons in these chains are scheduled after resolvable addons, then fail start with `Missing dependencies: ...` because required deps never reach `running`

---

## Custom Admin Permissions

Addons can define their own permission strings that integrate with fxPanel's existing permission system. These appear in the admin role editor alongside built-in permissions.

### Defining Permissions

Add an `adminPermissions` array to your `addon.json`:

```json
{
  "adminPermissions": [
    {
      "id": "my-addon.manage",
      "label": "Manage My Addon",
      "description": "Allows access to addon configuration"
    },
    {
      "id": "my-addon.view-reports",
      "label": "View Reports",
      "description": "Allows viewing addon-generated reports"
    }
  ]
}
```

### How They Work

- Custom permissions appear in the **Permissions** tab of the admin role editor under an "Addon Permissions" group
- They follow the same grant/deny model as built-in permissions
- The `all_permissions` role automatically includes all custom permissions

### Checking in Routes

```js
addon.registerRoute('POST', '/config', async (req) => {
  if (!req.admin.hasPermission('my-addon.manage')) {
    return { status: 403, body: { error: 'Insufficient permissions' } };
  }
  // ... update config ...
  return { status: 200, body: { success: true } };
});
```

### Widget/Page Permissions

You can reference your custom permission IDs in the manifest's `permission` fields for pages and widgets:

```json
{
  "panel": {
    "pages": [
      {
        "path": "/reports",
        "title": "Reports",
        "component": "ReportsPage",
        "permission": "my-addon.view-reports"
      }
    ]
  }
}
```

---

## Settings Component

Addons can provide a settings UI that appears in the fxPanel Settings page under the addon's tab.

### Declaring Settings

Add `settingsComponent` to your panel manifest in `addon.json`:

```json
{
  "panel": {
    "entry": "panel/index.js",
    "settingsComponent": "MySettingsComponent"
  }
}
```

### Exporting the Component

Your panel entry must export a React component matching the declared name:

```js
// panel/index.js
export function MySettingsComponent() {
  const [webhookUrl, setWebhookUrl] = React.useState('');

  const save = async () => {
    const api = globalThis.txAddonApi;
    await fetch('/addons/my-addon/api/settings', {
      method: 'POST',
      credentials: 'same-origin',
      headers: api.getHeaders(),
      body: JSON.stringify({ webhookUrl }),
    });
  };

  return React.createElement('div', null,
    React.createElement('label', null, 'Webhook URL'),
    React.createElement('input', {
      value: webhookUrl,
      onChange: (e) => setWebhookUrl(e.target.value),
    }),
    React.createElement('button', { onClick: save }, 'Save')
  );
}
```

The settings component renders inside the Settings page in its own tab, automatically labeled with the addon's display name.

---

## Addon Log Viewer

fxPanel provides a built-in log viewer for addon logs in the panel. Admins can view real-time and historical logs for each addon.

### How Logging Works

All calls to `addon.log.info()`, `addon.log.warn()`, and `addon.log.error()` are:

1. Printed to fxPanel's server console with the `[addon:<id>]` prefix
2. Stored in a circular buffer (last 1000 entries per addon)
3. Available via the Addons page log viewer in the panel

### API Endpoint

Addon logs can be retrieved programmatically:

```
GET /addons/<id>/logs?level=info&limit=100
```

Returns the most recent log entries filtered by level. Requires `all_permissions`.

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

> ⛔ **Do not:**
> - Shell out / spawn child processes from your addon
> - Attempt to read files outside your addon directory
> - Store sensitive data (tokens, passwords) in addon storage — it's a plain JSON file on disk

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

### Public routes return "Not found"

- Ensure `"publicRoutes": true` is set in your `addon.json` manifest
- Verify your route paths use simple `:param` segments — regex patterns like `:path(.*)` are not supported by the SDK matcher
- Check that the number of path segments in the request matches the registered route pattern

### Dedicated public port not starting

- Check if another process is already using the configured port
- Verify the port number is valid (1024–65535)
- Check the console for `[addon:<id>]` bind errors
- If no port is configured, routes are available via the `/site/<addonId>/` fallback on the main port

---

## Example: WebHost Addon

The `addon-watchdog` directory ships with fxPanel as a reference, but the **WebHost** addon demonstrates the public routes system. It provides:

- A configurable home page served on a dedicated public port
- Dual routing pattern: a default HTML template for `/` and static asset serving for `/assets/:file`
- Admin-configurable settings via the panel (server name, description, accent color)
- Integration with the settings component system

### Manifest

```json
{
  "id": "addon-webhost",
  "name": "WebHost",
  "description": "Serve a public website for your FiveM/RedM server",
  "version": "1.0.0",
  "author": "fxPanel",
  "fxpanel": {
    "minVersion": "0.2.1-Beta"
  },
  "publicRoutes": true,
  "publicServer": {
    "defaultPort": 8080
  },
  "permissions": {
    "required": ["storage"],
    "optional": []
  },
  "server": {
    "entry": "server/index.js"
  },
  "settings": {
    "component": "WebHostSettings"
  }
}
```

### Dual Route Pattern

```js
import { createAddon } from 'addon-sdk';

const addon = createAddon();

// Serve the home page
addon.registerPublicRoute('GET', '/', async (req) => {
  const config = await addon.storage.get('config') || {};
  const html = buildTemplate(config);
  return {
    status: 200,
    headers: { 'Content-Type': 'text/html' },
    body: html,
  };
});

// Serve static assets
addon.registerPublicRoute('GET', '/assets/:file', async (req) => {
  // ... read and serve file from assets directory ...
});

addon.ready();
```

### Default Theme

WebHost ships with a modern default template featuring a gradient background, glassmorphism cards, animated stats counters, and a fully responsive layout. Server administrators can customize the server name, description, and accent color through the Settings page.

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

---

## Quick Reference

### SDK API Cheatsheet

```js
import { createAddon } from 'addon-sdk';
const addon = createAddon();

// Identity
addon.id;                          // string — addon ID
addon.permissions;                 // string[] — granted permissions

// Routes
addon.registerRoute('GET', '/path/:param', handler);
addon.registerPublicRoute('GET', '/path/*', handler);  // wildcard catch-all

// Storage
await addon.storage.get(key);             // get value
await addon.storage.getOr(key, default);  // get with fallback
await addon.storage.has(key);             // check existence
await addon.storage.set(key, value);      // set value
await addon.storage.delete(key);          // delete key
await addon.storage.list(prefix?);        // list keys

// Events
addon.on('playerJoining', handler);
addon.off('playerJoining', handler);      // remove handler
addon.off('playerJoining');               // remove all handlers

// WebSocket
addon.ws.push(event, data);
addon.ws.onSubscribe(handler);
addon.ws.onUnsubscribe(handler);

// Players
await addon.players.addTag(netid, tagId);
await addon.players.removeTag(netid, tagId);

// Logging
addon.log.info(msg);
addon.log.warn(msg);
addon.log.error(msg);

// Lifecycle
addon.ready();
```

### Manifest Quick Template

```json
{
  "id": "my-addon",
  "name": "My Addon",
  "description": "What it does",
  "version": "1.0.0",
  "author": "YourName",
  "fxpanel": { "minVersion": "0.2.1-Beta" },
  "permissions": { "required": ["storage"], "optional": [] },
  "server": { "entry": "server/index.js" },
  "panel": {
    "entry": "panel/index.js",
    "pages": [{ "path": "/my-page", "title": "My Page", "component": "MyPage", "sidebar": true }],
    "widgets": [{ "slot": "dashboard.main", "component": "MyWidget", "title": "My Widget" }]
  }
}
```

### Route Handler Signatures

```js
// Authenticated route
(req) => {
  req.method;         // 'GET', 'POST', etc.
  req.path;           // matched path
  req.params;         // { param: 'value', '*': 'catch/all/path' }
  req.headers;        // HTTP headers
  req.body;           // parsed JSON body
  req.admin.name;     // admin name
  req.admin.hasPermission('perm');
  return { status: 200, headers: {}, body: {} };
}

// Public route (req.admin is always null)
(req) => { return { status: 200, body: 'ok' }; }
```

---

## Distribution

### Before You Submit

- Test your addon on a clean fxPanel install
- Ensure all required permissions are documented
- Verify the addon works with the declared `fxpanel.minVersion`
- Include a README with setup instructions
- Test hot-reload behavior if applicable

### Submission Format

Package your addon as a `.zip` containing the addon directory:

```
my-addon.zip
└── my-addon/
    ├── addon.json
    ├── package.json
    ├── server/
    │   └── index.js
    └── ...
```

The user extracts it into `addons/` so that `addons/my-addon/addon.json` exists at the expected location.
