import { resolve } from 'node:path'
import Fastify from 'fastify'
import reactify from '@cyb3rcore/reactify'
import * as renderer from '@cyb3rcore/reactify/renderer'

const server = Fastify({
  logger: {
    transport: { target: '@fastify/one-line-logger' },
  },
})

await server.register(reactify, {
  root: resolve(import.meta.dirname, '..'),
  dev: process.argv.includes('--dev'),
  renderer,
})

await server.vite.ready()
await server.listen({ port: 3000 })
