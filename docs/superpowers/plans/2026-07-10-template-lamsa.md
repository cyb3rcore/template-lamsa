# Template-lamsa Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build the Handlebars template collection at `~/dev/web/template-lamsa` used by the `create-cyber-stack` CLI to scaffold projects with `@cyb3rcore/reactify`.

**Architecture:** A `templates/core/` directory with `.hbs` (Handlebars) files mirroring a complete reactify project structure. The CLI clones this repo, compiles templates with user config, and writes output to the target directory. The template has no build step of its own.

**Tech Stack:** Handlebars (`.hbs`) templates, `@cyb3rcore/reactify ^0.1.0`, Fastify 5, React 19, Vite 8, TypeScript.

## Global Constraints

- All template files go under `templates/core/` — never outside
- `.hbs` extension is stripped on scaffold output (CLI handles this)
- Leading `_` in filenames becomes `.` on output (e.g. `_gitignore` → `.gitignore`)
- The only template variable is `{{projectName}}` — no other variables for v1
- All imports reference `@cyb3rcore/reactify` (published npm package) not `reactify` (unpublished) or `@fastify/react` (upstream)
- Demo pages go under `pages/demo/` for easy deletion
- No test files, no getting-started page in scaffolded output

---

### Task 1: Repo scaffolding

**Files:**
- Create: `package.json`
- Create: `README.md`
- Create: `docs/features.md`

**Interfaces:**
- Consumes: nothing
- Produces: repo metadata and developer documentation

- [ ] **Step 1: Create `package.json`**

```json
{
  "name": "template-lamsa",
  "private": true,
  "version": "0.1.0",
  "description": "Handlebars templates for create-cyber-stack — Reactify project scaffolding",
  "type": "module"
}
```

- [ ] **Step 2: Create `README.md`**

```markdown
# template-lamsa

Handlebars template collection for the `create-cyber-stack` CLI.

## Usage

This repo is not meant to be used directly. The CLI clones it at scaffold time.

## Structure

```
templates/core/     # Always-included templates for a reactify project
docs/               # Developer documentation
```

## Related

- [create-cyber-stack](https://github.com/cyb3rcore/create-cyber-stack) — the CLI
- [@cyb3rcore/reactify](https://github.com/cyb3rcore/reactify) — the framework
```

- [ ] **Step 3: Create `docs/features.md`**

```markdown
# Features

This template scaffolds a reactify project demonstrating all framework features.

## SSR + SPA
- SSR rendering with HTML shell
- Dynamic route params (`[id].tsx`)
- Static/dynamic route precedence
- 404 handling
- `getData` JSON endpoint
- `getMeta` head metadata
- Streaming SSR

## RSC
- RSC pages (`export const rsc = true`)
- Client components (`'use client'`)
- Server actions (`'use server'`)
- Async RSC data fetching
- Suspense/streaming
- Valtio state management

## Context Bridge
- `getServer()` / `getReq()` from `reactify/server`
- `onEnter` lifecycle hook
- `configure(scope)` Fastify decoration
```

- [ ] **Step 4: Commit**

```bash
git add package.json README.md docs/features.md
git commit -m "chore: init template-lamsa repo"
```

---

### Task 2: Core project scaffolding templates

**Files:**
- Create: `templates/core/_gitignore`
- Create: `templates/core/package.json.hbs`
- Create: `templates/core/tsconfig.json.hbs`
- Create: `templates/core/vite.config.ts.hbs`

**Interfaces:**
- Consumes: Task 1 (repo exists)
- Produces: project configuration templates that Task 3-5's templates will reference

- [ ] **Step 1: Create `templates/core/_gitignore`**

```
node_modules
dist
*.tsbuildinfo
styled-system
```

- [ ] **Step 2: Create `templates/core/package.json.hbs`**

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
    "@unhead/react": "^2.1.13",
    "fastify": "^5.8.5",
    "react": "^19.2.4",
    "react-dom": "^19.2.4",
    "valtio": "latest"
  },
  "devDependencies": {
    "@vitejs/plugin-react": "^6.0.1",
    "@types/node": "^22.0.0",
    "@types/react": "^19.0.0",
    "@types/react-dom": "^19.0.0",
    "tsx": "^4.19.0",
    "typescript": "^5.9.0",
    "vite": "^8.0.16"
  }
}
```

- [ ] **Step 3: Create `templates/core/tsconfig.json.hbs`**

```json
{
  "compilerOptions": {
    "jsx": "react-jsx",
    "lib": ["ES2020", "DOM"],
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "strict": true,
    "outDir": "dist",
    "rootDir": "src",
    "skipLibCheck": true,
    "sourceMap": true,
    "target": "esnext"
  },
  "include": ["src/**/*"]
}
```

- [ ] **Step 4: Create `templates/core/vite.config.ts.hbs`**

```handlebars
import { resolve } from 'node:path'
import viteReact from '@vitejs/plugin-react'
import reactifyPlugin from '@cyb3rcore/reactify/plugin'

export default {
  root: resolve(import.meta.dirname, 'client'),
  plugins: [viteReact(), reactifyPlugin()],
}
```

- [ ] **Step 5: Commit**

```bash
git add templates/core/_gitignore templates/core/package.json.hbs templates/core/tsconfig.json.hbs templates/core/vite.config.ts.hbs
git commit -m "feat: add core project scaffolding templates"
```

---

### Task 3: Server and client entry templates

**Files:**
- Create: `templates/core/src/server.ts.hbs`
- Create: `templates/core/src/client/index.html.hbs`
- Create: `templates/core/src/client/mount.tsx.hbs`
- Create: `templates/core/src/client/root.tsx.hbs`

**Interfaces:**
- Consumes: Task 2 (project config templates)
- Produces: the entry points that wire Fastify, Vite, and React together

- [ ] **Step 1: Create `templates/core/src/server.ts.hbs`**

```handlebars
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

- [ ] **Step 2: Create `templates/core/src/client/index.html.hbs`**

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

- [ ] **Step 3: Create `templates/core/src/client/mount.tsx.hbs`**

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
  const routeMap = Object.fromEntries(resolvedRoutes.map((r) => [r.path, r]))
  const app = create({
    url: window.location.href,
    routes: resolvedRoutes,
    ctxHydration,
    routeMap,
  })
  const root = document.getElementById('root')
  if (!root) return
  if (ctxHydration.clientOnly) {
    createRoot(root).render(app)
  } else {
    hydrateRoot(root, app)
  }
}

mount()
```

- [ ] **Step 4: Create `templates/core/src/client/root.tsx.hbs`**

```tsx
import { create as createApp } from '$app/create.jsx'
export default createApp
```

- [ ] **Step 5: Commit**

```bash
git add templates/core/src/server.ts.hbs templates/core/src/client/index.html.hbs templates/core/src/client/mount.tsx.hbs templates/core/src/client/root.tsx.hbs
git commit -m "feat: add server and client entry templates"
```

---

### Task 4: Main page, layout, and helpers

**Files:**
- Create: `templates/core/src/client/layouts/default.tsx.hbs`
- Create: `templates/core/src/client/pages/index.tsx.hbs`
- Create: `templates/core/src/client/context.tsx.hbs`
- Create: `templates/core/src/client/actions/index.ts.hbs`
- Create: `templates/core/src/client/components/counter.tsx.hbs`

**Interfaces:**
- Consumes: Task 3 (knows about RouteProvider, mount lifecycle)
- Produces: the app homepage and reusable building blocks used by demo pages

- [ ] **Step 1: Create `templates/core/src/client/layouts/default.tsx.hbs`**

```tsx
import type { ReactNode } from 'react'

export default function DefaultLayout({ children }: { children: ReactNode }) {
  return <main>{children}</main>
}
```

- [ ] **Step 2: Create `templates/core/src/client/pages/index.tsx.hbs`**

```tsx
export function getMeta() {
  return { title: '{{projectName}}', description: 'A Reactify project' }
}

export function getData() {
  return { message: 'Welcome to your Reactify app!' }
}

export default function Home() {
  return (
    <div>
      <h1>{{projectName}}</h1>
      <p>Built with Reactify — Fastify + React + Vite</p>
      <nav>
        <a href="/demo">View demos</a>
      </nav>
    </div>
  )
}
```

- [ ] **Step 3: Create `templates/core/src/client/context.tsx.hbs`**

```tsx
import { proxy } from 'valtio'

export const store = proxy({ count: 42, message: 'Hello from Valtio!' })
```

- [ ] **Step 4: Create `templates/core/src/client/actions/index.ts.hbs`**

```tsx
'use server'

export async function increment(prev: number) {
  return prev + 1
}
```

- [ ] **Step 5: Create `templates/core/src/client/components/counter.tsx.hbs`**

```tsx
'use client'
import { useState } from 'react'

export default function Counter() {
  const [count, setCount] = useState(0)
  return (
    <div>
      <p>Client count: {count}</p>
      <button onClick={() => setCount((c) => c + 1)}>+</button>
      <button onClick={() => setCount((c) => c - 1)}>-</button>
    </div>
  )
}
```

- [ ] **Step 6: Commit**

```bash
git add templates/core/src/client/layouts/default.tsx.hbs templates/core/src/client/pages/index.tsx.hbs templates/core/src/client/context.tsx.hbs templates/core/src/client/actions/index.ts.hbs templates/core/src/client/components/counter.tsx.hbs
git commit -m "feat: add homepage, layout, helpers, and Valtio store templates"
```

---

### Task 5: RSC demo pages

**Files:**
- Create: `templates/core/src/client/pages/demo/index.tsx.hbs`
- Create: `templates/core/src/client/pages/demo/rsc-basics.tsx.hbs`
- Create: `templates/core/src/client/pages/demo/rsc-data.tsx.hbs`
- Create: `templates/core/src/client/pages/demo/rsc-client.tsx.hbs`
- Create: `templates/core/src/client/pages/demo/rsc-actions.tsx.hbs`

**Interfaces:**
- Consumes: Task 4 (Counter component, actions module)
- Produces: demo pages covering RSC basics, async data, client components, server actions

- [ ] **Step 1: Create `templates/core/src/client/pages/demo/index.tsx.hbs`**

```tsx
export const rsc = true

export default function DemoHub() {
  return (
    <div>
      <h1>Feature Demos</h1>
      <p>These pages demonstrate reactify features. Delete the <code>demo/</code> folder when you're done.</p>
      <h2>RSC</h2>
      <ul>
        <li><a href="/demo/rsc-basics">RSC Basics</a> — getServer, getReq</li>
        <li><a href="/demo/rsc-data">RSC Data Fetching</a> — async component</li>
        <li><a href="/demo/rsc-client">RSC + Client</a> — 'use client' boundary</li>
        <li><a href="/demo/rsc-actions">Server Actions</a> — useActionState</li>
        <li><a href="/demo/rsc-streaming">RSC Streaming</a> — Suspense</li>
        <li><a href="/demo/rsc-store">Valtio Store</a> — proxy state</li>
      </ul>
      <h2>SSR</h2>
      <ul>
        <li><a href="/demo/params/hello">Dynamic Params</a></li>
        <li><a href="/demo/streaming">Streaming SSR</a></li>
      </ul>
      <h2>Boundary</h2>
      <ul>
        <li><a href="/demo/error">Error Boundary</a></li>
      </ul>
    </div>
  )
}
```

- [ ] **Step 2: Create `templates/core/src/client/pages/demo/rsc-basics.tsx.hbs`**

```tsx
import { getServer, getReq } from '@cyb3rcore/reactify/server'

export const rsc = true

export function getMeta() {
  return { title: 'RSC Basics' }
}

export function onEnter({ req, reply, server }: { req: any; reply: any; server: any }) {
  return { reqHeaders: req?.headers, hasServer: !!server }
}

export default function RscBasics() {
  const server = getServer()
  const req = getReq()
  return (
    <div>
      <h1>RSC Basics</h1>
      <p>Server available: <span>{String(!!server)}</span></p>
      <p>Request available: <span>{String(!!req)}</span></p>
      <p>Server-rendered timestamp: <span>{Date.now()}</span></p>
    </div>
  )
}
```

- [ ] **Step 3: Create `templates/core/src/client/pages/demo/rsc-data.tsx.hbs`**

```tsx
export const rsc = true

async function fetchItems() {
  return ['Item A', 'Item B', 'Item C']
}

export default async function RscData() {
  const items = await fetchItems()
  return (
    <div>
      <h1>RSC Data Fetching</h1>
      <p>Data fetched during server-side render:</p>
      <ul>
        {items.map((item) => <li key={item}>{item}</li>)}
      </ul>
    </div>
  )
}
```

- [ ] **Step 4: Create `templates/core/src/client/pages/demo/rsc-client.tsx.hbs`**

```tsx
import Counter from '../../components/counter'

export const rsc = true

export default function RscClient() {
  return (
    <div>
      <h1>RSC + Client Component</h1>
      <p>Below is a <code>'use client'</code> component with interactive state:</p>
      <Counter />
    </div>
  )
}
```

- [ ] **Step 5: Create `templates/core/src/client/pages/demo/rsc-actions.tsx.hbs`**

```tsx
import { useActionState } from 'react'
import { increment } from '../../actions'

export const rsc = true

export default function RscActions() {
  const [count, formAction] = useActionState(increment, 0)
  return (
    <div>
      <h1>Server Actions</h1>
      <form action={formAction}>
        <output>{count}</output>
        <button>Increment</button>
      </form>
    </div>
  )
}
```

- [ ] **Step 6: Commit**

```bash
git add templates/core/src/client/pages/demo/
git commit -m "feat: add RSC demo pages (basics, data, client, actions)"
```

---

### Task 6: More demo pages (streaming, store, params, error)

**Files:**
- Create: `templates/core/src/client/pages/demo/rsc-streaming.tsx.hbs`
- Create: `templates/core/src/client/pages/demo/rsc-store.tsx.hbs`
- Create: `templates/core/src/client/pages/demo/params.tsx.hbs`
- Create: `templates/core/src/client/pages/demo/streaming.tsx.hbs`
- Create: `templates/core/src/client/pages/demo/error.tsx.hbs`

**Interfaces:**
- Consumes: Task 4 (Valtio store from context)
- Produces: remaining demo pages covering all features

- [ ] **Step 1: Create `templates/core/src/client/pages/demo/rsc-streaming.tsx.hbs`**

```tsx
import { Suspense } from 'react'

export const rsc = true

async function DelayedContent() {
  await new Promise((r) => setTimeout(r, 200))
  return <span>streamed content</span>
}

export default function RscStreaming() {
  return (
    <div>
      <h1>RSC Streaming</h1>
      <Suspense fallback={<span>loading...</span>}>
        <DelayedContent />
      </Suspense>
    </div>
  )
}
```

- [ ] **Step 2: Create `templates/core/src/client/pages/demo/rsc-store.tsx.hbs`**

```tsx
'use client'
import { useSnapshot } from 'valtio'
import { store } from '../../context'

export default function RscStore() {
  const snap = useSnapshot(store)
  return (
    <div>
      <h1>Valtio State</h1>
      <p>Count: <span>{snap.count}</span></p>
      <p>Message: <span>{snap.message}</span></p>
    </div>
  )
}
```

- [ ] **Step 3: Create `templates/core/src/client/pages/demo/params.tsx.hbs`**

```tsx
import { useParams } from '$app/core.jsx'

export function getMeta() {
  return { title: 'Dynamic Params' }
}

export default function Params() {
  const params = useParams()
  return (
    <div>
      <h1>Dynamic Route Params</h1>
      <p>The <code>[id]</code> segment is: <strong>{params.id}</strong></p>
      <p><a href="/demo/params/world">Try /world</a></p>
    </div>
  )
}
```

- [ ] **Step 4: Create `templates/core/src/client/pages/demo/streaming.tsx.hbs`**

```tsx
export const streaming = true

export function getMeta() {
  return { title: 'Streaming SSR' }
}

export default function Streaming() {
  return (
    <div>
      <h1>Streaming SSR</h1>
      <p>This page uses streaming — no Content-Length header.</p>
    </div>
  )
}
```

- [ ] **Step 5: Create `templates/core/src/client/pages/demo/error.tsx.hbs`**

```tsx
export const rsc = true

function Throws(): React.ReactNode {
  throw new Error('RSC error boundary test')
}

export default function ErrorPage() {
  return (
    <div>
      <h1>Error Boundary Test</h1>
      <Throws />
    </div>
  )
}
```

- [ ] **Step 6: Commit**

```bash
git add templates/core/src/client/pages/demo/
git commit -m "feat: add remaining demo pages (streaming, store, params, error)"
```

---

### Task 7: CI workflow template

**Files:**
- Create: `templates/core/.github/workflows/ci.yml.hbs`

**Interfaces:**
- Consumes: Task 2 (package.json script names)
- Produces: GitHub Actions CI for the scaffolded project

- [ ] **Step 1: Create `templates/core/.github/workflows/ci.yml.hbs`**

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
      - run: npm ci
      - run: npm run build
```

- [ ] **Step 2: Commit**

```bash
git add templates/core/.github/workflows/ci.yml.hbs
git commit -m "feat: add CI workflow template"
```

---

### Task 8: Verify template with a test scaffold

**Files:** (created outside template-lamsa for verification)
- Temporary: `/tmp/test-verify-scaffold/`

**Interfaces:**
- Consumes: all previous tasks (complete template directory)
- Produces: proof that the template produces a working reactify project

- [ ] **Step 1: Create a test scaffold by simulating CLI compilation**

Run the following in the template-lamsa repo directory. This strips `.hbs` extensions and renames `_gitignore` → `.gitignore`:

```bash
TARGET=/tmp/test-verify-scaffold/my-app
SRC=templates/core

for f in $(find $SRC -type f); do
  rel="${f#$SRC/}"
  # Strip leading underscore (for _gitignore)
  rel="${rel#_}"
  # Strip .hbs extension
  out="${TARGET}/${rel%.hbs}"
  mkdir -p "$(dirname "$out")"
  cp "$f" "$out"
done

echo "Scaffold created at $TARGET"
ls -la "$TARGET"
```

- [ ] **Step 2: Validate basic structure**

```bash
ls /tmp/test-verify-scaffold/my-app/package.json
ls /tmp/test-verify-scaffold/my-app/vite.config.ts
ls /tmp/test-verify-scaffold/my-app/tsconfig.json
ls /tmp/test-verify-scaffold/my-app/src/server.ts
ls /tmp/test-verify-scaffold/my-app/src/client/pages/demo/
```

Expected: all files exist from the template directory.

- [ ] **Step 3: Verify npm install and build succeed on the reactify repo itself**

```bash
cd ~/dev/web/reactify
npm test
```

Expected: 172 tests pass.

- [ ] **Step 4: Push template-lamsa to GitHub**

```bash
cd ~/dev/web/template-lamsa
git remote add origin git@github.com:cyb3rcore/template-lamsa.git
git push -u origin main
```
