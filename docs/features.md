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
