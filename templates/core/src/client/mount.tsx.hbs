import { hydrateRoot, createRoot } from 'react-dom/client'
import { createElement, useState, useEffect, startTransition } from 'react'
import { hydrateRoutes } from '@cyb3rcore/reactify/client'
import { createHead } from '@unhead/react/client'
import routes from '$app/routes.js'
import create from '$app/create.jsx'
import * as context from '$app/context.js'

async function mount() {
  const root = document.getElementById('root')
  if (!root) return

  // ---- RSC path: detect flight data injected by RSC→SSR pipeline ----
  if (typeof window !== 'undefined' && window.__FLIGHT_DATA) {
    const { rscStream } = await import('rsc-html-stream/client')
    const { createFromReadableStream, setServerCallback } =
      await import('@vitejs/plugin-rsc/browser')

    // Polyfill __webpack_require__ for RSC vendor module loading.
    // The react-server-dom vendor uses __webpack_require__ but Vite's esbuild
    // pre-bundling skips the patch transform, leaving it undefined.
    const wpRequire = '__' + 'webpack_require' + '__'
    if (typeof (globalThis as any)[wpRequire] === 'undefined') {
      ;(globalThis as any)[wpRequire] = (id: string) => {
        if (id.includes('$$cache='))
          return (globalThis as any).__vite_rsc_require__(id)
        const cc = '$' + 'cache='
        const cleanId = id.includes(cc) ? id.split(cc)[0] : id
        return (globalThis as any).__vite_rsc_require__(cleanId)
      }
      ;(globalThis as any)[wpRequire].u = () => {}
    }

    // Also strip $cache= in __vite_rsc_require__ directly (bypasses polyfill)
    const _orig = (globalThis as any).__vite_rsc_require__
    if (_orig) {
      ;(globalThis as any).__vite_rsc_require__ = (id: string) => {
        if (id.includes('$$cache=')) return _orig(id)
        const idx = id.indexOf('$' + 'cache=')
        if (idx !== -1) id = id.slice(0, idx)
        return _orig(id)
      }
    }

    const initialPayload = await createFromReadableStream(rscStream)

    function RscRoot() {
      const [payload, setPayload] = useState(initialPayload)

      useEffect(() => {
        window.__rscSetPayload = (v: unknown) =>
          startTransition(() => setPayload(v))
        setServerCallback(async (id: string, args: unknown[]) => {
          const { createTemporaryReferenceSet, encodeReply, createFromFetch } =
            await import('@vitejs/plugin-rsc/browser')
          const temporaryReferences = createTemporaryReferenceSet()
          const rscUrl = `${window.location.pathname}_.rsc${window.location.search}`
          const payload = await createFromFetch(
            fetch(rscUrl, {
              method: 'POST',
              headers: { 'x-rsc-action': id },
              body: await encodeReply(args, { temporaryReferences }),
            }),
            { temporaryReferences },
          )
          startTransition(() => setPayload(payload))
          const { ok, data } = (payload as any).returnValue ?? {}
          if (!ok) throw data
          return data
        })
      }, [])

      useEffect(() => {
        if ((payload as any).head?.title) {
          document.title = (payload as any).head.title
        }
      }, [payload])

      return (payload as any).matches?.[0]?.element ?? null
    }

    hydrateRoot(root, createElement(RscRoot), {
      formState: (initialPayload as any).formState,
    })
    return
  }

  // ---- Non-RSC path ----
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
  if (ctxHydration.clientOnly) {
    createRoot(root).render(app)
  } else {
    hydrateRoot(root, app)
  }
}

mount()
