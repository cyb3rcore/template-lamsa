# template-lamsa — Design Specification

**Date:** 2026-07-10
**Status:** Draft

## Overview

template-lamsa is the Handlebars template collection used by the `create-cyber-stack` CLI to scaffold new projects using `@cyb3rcore/reactify` (a Fastify + Vite + React SSR framework with RSC support).

It replaces the previous `template-amal` which targeted the upstream `@fastify/react` stack.

## Repository

- **Path:** `~/dev/web/template-lamsa`
- **Purpose:** Template source — the CLI clones this repo at scaffold time, compiles `.hbs` templates with the user's project config, and writes the output to the target directory.
- **Not a standalone project:** Cannot be run directly. It is consumed by the CLI.

## Features Covered

Every scaffolded project demonstrates all 18 features tested across the 4 e2e fixtures in the reactify repo:

### SSR + SPA (react-base)
1. SSR rendering with HTML shell
2. Dynamic route params (`[id].tsx`)
3. Static/dynamic route precedence
4. 404 handling
5. `getData` JSON endpoint (`/-/data/*`)
6. `getMeta` head metadata
7. Streaming SSR (no Content-Length)

### RSC (react-rsc)
8. RSC pages (`export const rsc = true`)
9. Client components with `'use client'`
10. Server actions (`'use server'` + `useActionState`)
11. Async RSC data fetching
12. Suspense streaming in RSC
13. Valtio state management
14. Error boundaries in RSC

### Context Bridge (react-context)
15. `getServer()` from `reactify/server`
16. `getReq()` from `reactify/server`
17. `onEnter` lifecycle hook
18. `configure(scope)` Fastify decoration hook

## Directory Structure

```
template-lamsa/
├── README.md              # Repo metadata, usage instructions
├── package.json           # Template package metadata
├── docs/
│   └── features.md        # Feature reference for repo navigators
└── templates/
    └── core/              # Always included — no optional addons yet
        ├── _gitignore              # → .gitignore (leading _ stripped)
        ├── package.json.hbs        # Project dependencies
        ├── tsconfig.json.hbs       # Strict TypeScript config
        ├── vite.config.ts.hbs      # Vite + reactify/plugin
        ├── src/
        │   ├── server.ts.hbs       # Fastify server entry
        │   └── client/
        │       ├── index.html.hbs  # HTML shell with <!-- element --> marker
        │       ├── mount.tsx.hbs   # Client hydration entry
        │       ├── root.tsx.hbs    # RouteProvider + RouteRenderer
        │       ├── components/
        │       │   └── counter.tsx.hbs   # 'use client' Counter
        │       ├── actions/
        │       │   └── index.ts.hbs      # 'use server' actions
        │       ├── pages/
        │       │   ├── index.tsx.hbs     # App homepage (user keeps)
        │       │   └── demo/             # 🗑️ Delete to clean up
        │       │       ├── index.tsx.hbs
        │       │       ├── rsc-basics.tsx.hbs
        │       │       ├── rsc-data.tsx.hbs
        │       │       ├── rsc-client.tsx.hbs
        │       │       ├── rsc-actions.tsx.hbs
        │       │       ├── rsc-streaming.tsx.hbs
        │       │       ├── rsc-store.tsx.hbs
        │       │       ├── params.tsx.hbs
        │       │       ├── streaming.tsx.hbs
        │       │       └── error.tsx.hbs
        │       └── layouts/
        │           └── default.tsx.hbs
        └── .github/
            └── workflows/
                └── ci.yml.hbs
```

## Template Details

### package.json.hbs

```handlebars
{
  "name": "{{projectName}}",
  "private": true,
  "type": "module",
  "scripts": {
    "dev": "tsx src/server.ts --dev",
    "start": "NODE_ENV=production node src/server.js",
    "build": "vite build --ssrManifest && vite build --ssr",
    "clean": "rm -rf dist"
  },
  "dependencies": {
    "@cyb3rcore/reactify": "^0.1.0",
    "fastify": "^5.8.5",
    "react": "^19.2.4",
    "react-dom": "^19.2.4",
    "valtio": "latest",
    "@unhead/react": "^2.1.13"
  },
  "devDependencies": {
    "@vitejs/plugin-react": "^6.0.1",
    "@types/node": "^22.0.0",
    "@types/react": "^19.0.0",
    "@types/react-dom": "^19.0.0",
    "typescript": "^5.9.0",
    "vite": "^8.0.16",
    "tsx": "^4.19.0"
  }
}
```

- `@cyb3rcore/reactify` is the framework dependency (published on npm)
- React 19, Fastify 5, Vite 8 — matching the framework's peer deps
- `tsx` for dev server runner (TypeScript execution)
- Vite's `--ssrManifest` and `--ssr` flags for production builds

### server.ts.hbs

```
import { resolve } from 'node:path'
import Fastify from 'fastify'
import reactify from '@cyb3rcore/reactify'
import * as renderer from '@cyb3rcore/reactify/renderer'

const server = Fastify({ logger: true })

await server.register(reactify, {
  root: resolve(import.meta.dirname, '..'),
  dev: process.argv.includes('--dev'),
  renderer,
})

await server.vite.ready()
await server.listen({ port: 3000 })
```

Pattern matches the e2e fixtures exactly — clean, no excess config.

### vite.config.ts.hbs

```
import { resolve } from 'node:path'
import viteReact from '@vitejs/plugin-react'
import reactifyPlugin from '@cyb3rcore/reactify/plugin'

export default {
  root: resolve(import.meta.dirname, 'client'),
  plugins: [viteReact(), reactifyPlugin()],
}
```

### Demo Pages

Each demo page exercises one or more features:

| Route                   | Template file               | Features                                 |
| ----------------------- | --------------------------- | ---------------------------------------- |
| `/demo`                   | `demo/index.tsx.hbs`          | Hub page with navigation to all demos    |
| `/demo/rsc-basics`        | `demo/rsc-basics.tsx.hbs`     | `export const rsc = true`, `getServer()` |
| `/demo/rsc-data`          | `demo/rsc-data.tsx.hbs`       | Async RSC data fetch                     |
| `/demo/rsc-client`        | `demo/rsc-client.tsx.hbs`     | `'use client'` Counter component         |
| `/demo/rsc-actions`       | `demo/rsc-actions.tsx.hbs`    | Server action with `useActionState`      |
| `/demo/rsc-streaming`     | `demo/rsc-streaming.tsx.hbs`  | `<Suspense>` with delayed content        |
| `/demo/rsc-store`         | `demo/rsc-store.tsx.hbs`      | Valtio `proxy()` + `useSnapshot`         |
| `/demo/params/hello`      | `demo/params.tsx.hbs`         | Dynamic route params `[id]`              |
| `/demo/streaming`         | `demo/streaming.tsx.hbs`      | Non-RSC streaming SSR                    |
| `/demo/error`             | `demo/error.tsx.hbs`          | RSC error boundary                       |

### index.html.hbs

Minimal shell with the `<!-- element -->` marker for template splitting:

```handlebars
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
  </head>
  <body>
    <div id="root"><!-- element --></div>
    <script type="module" src="/$app/mount.ts"></script>
  </body>
</html>
```

### mount.tsx.hbs

Client hydration entry. Pattern taken from the `@fastify/react` upstream but adapted to reactify's virtual module layout:

```tsx
import { hydrateRoot, createRoot } from 'react-dom/client'
import { hydrateRoutes } from '@cyb3rcore/reactify/client'
import { createHead } from '@unhead/react/client'
import routes from '$app/routes.js'
import create from '$app/create.jsx'
import * as context from '$app/context.js'

async function mount() {
  const ctxHydration = { ...window.route, ...context }
  const useHead = createHead()
  ctxHydration.useHead = useHead
  const resolvedRoutes = await hydrateRoutes(routes)
  const routeMap = Object.fromEntries(resolvedRoutes.map(r => [r.path, r]))
  const app = create({ url: window.location.href, routes: resolvedRoutes, ctxHydration, routeMap })
  const root = document.getElementById('root')
  if (root) {
    if (ctxHydration.clientOnly) {
      createRoot(root).render(app)
    } else {
      hydrateRoot(root, app)
    }
  }
}

mount()
```

### root.tsx.hbs

Uses reactify's `RouteProvider` + `RouteRenderer` (not react-router):

```tsx
import { create as createApp } from '$app/create.jsx'
```

This is a simple passthrough — the actual creation logic is in the virtual `create.tsx` module.

### context.tsx.hbs

```tsx
import { proxy } from 'valtio'

export const store = proxy({ count: 42, message: 'Hello from Valtio!' })
```

### Pages with getData/getMeta

The homepage (`pages/index.tsx.hbs`) exports both:

```tsx
export function getMeta() {
  return { title: '{{projectName}}', description: 'A Reactify project' }
}

export function getData() {
  return { message: 'Hello from the server!' }
}
```

## CLI Integration

The CLI at `~/dev/web/create-reactify-app` will reference this repo as its template source. The CLI's `template-fetcher.ts` clones `template-lamsa` instead of `template-amal`.

Template variables used:
- `{{projectName}}` — user-provided project name

No other template variables are needed for the v1 core template. Feature-based templates (API, auth, DB, addons) will be added later as CLI cherry-pick options.

## Non-Goals (v1)

- No CSS framework included (user adds their own)
- No API/endpoint scaffolding
- No authentication boilerplate
- No database setup
- No MCP/skills configuration
- No test files scaffolded
- No getting-started page scaffolded

These will be designed in a separate CLI cherry-pick session.
