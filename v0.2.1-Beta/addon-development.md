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