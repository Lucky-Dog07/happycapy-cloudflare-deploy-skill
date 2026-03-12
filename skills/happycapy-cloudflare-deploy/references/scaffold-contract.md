# Happycapy Cloudflare Scaffold Contract

Use this contract when deciding whether a project can be normalized and deployed by this skill.

## Target Stack

Prefer this stack:
- Vite
- React
- Tailwind CSS
- Hono
- Wrangler
- Workers for Platforms dynamic dispatch

Reason:
- The frontend can build to static assets.
- The backend can run inside the Workers runtime.
- The existing Happycapy app scaffold remains useful.
- A Dispatch Worker can expose the app through one stable platform URL.

## Minimum Project Shape

Expect these capabilities even if file names differ:
- A frontend that builds to a static output directory such as `dist/`
- A User Worker entrypoint for API and request handling, or enough code to generate one
- A Dispatcher Worker entrypoint, or enough code to generate one
- Wrangler configuration for the app Worker and the Dispatcher Worker
- Clear npm scripts for install and build

Prefer this script contract in `package.json`:

```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "deploy": "wrangler deploy"
  }
}
```

The existing `deploy` script may continue to represent the app Worker. Add Workers for Platforms-specific deploy commands only when needed.

## Preferred Routing Model

Use two Workers:
- A public Dispatch Worker on `workers.dev`
- A User Worker inside a Dispatch Namespace

Preferred request flow:
- The Dispatch Worker reads the first path segment as the User Worker name.
- The Dispatch Worker strips that segment from the forwarded request path.
- The User Worker serves static assets and `/api/*` routes.
- SPA routes fall back to `index.html`.

Prefer Dispatcher Wrangler configuration equivalent to:

```json
{
  "$schema": "./node_modules/wrangler/config-schema.json",
  "name": "hc-example-dispatcher",
  "main": "./src/dispatcher.ts",
  "compatibility_date": "2026-03-12",
  "workers_dev": true,
  "dispatch_namespaces": [
    {
      "binding": "DISPATCHER",
      "namespace": "hc-example-ns"
    }
  ]
}
```

Prefer User Worker Wrangler configuration equivalent to:

```json
{
  "$schema": "./node_modules/wrangler/config-schema.json",
  "main": "./src/worker.ts",
  "compatibility_date": "2026-03-12",
  "assets": {
    "directory": "./dist",
    "binding": "ASSETS",
    "not_found_handling": "single-page-application",
    "run_worker_first": ["/api/*"]
  }
}
```

Adjust paths to match the actual repo layout.

## Worker Entry Expectations

The User Worker entrypoint should:
- Handle `/api/*` routes with Hono or equivalent fetch routing
- Fall through to `env.ASSETS.fetch(request)` for frontend asset delivery when needed
- Avoid Node-only server assumptions such as Express, raw TCP listeners, or filesystem writes

The Dispatch Worker entrypoint should:
- Read a User Worker name from the incoming URL
- Resolve the target with `env.DISPATCHER.get(workerName)`
- Rewrite the pathname before forwarding when using path-based routing
- Return a clear error when the URL does not include a target worker

## Acceptable Variants

These are acceptable with small adaptation:
- Frontend-only Vite SPA with no API routes
- React + Vite app whose Worker file lives under `worker/` or `src/worker/`
- Hono-first app that already serves static assets via Cloudflare bindings
- A repo that already deploys as a regular Worker and only needs the Dispatch Worker layer

## Reject Or Migrate

Reject or migrate if you see:
- Next.js SSR assumptions that require unsupported Node features
- Express/Fastify/Koa servers expecting a long-running Node process
- Native modules that require platform-specific binaries
- Build output that depends on a traditional VM or container runtime

## Why This Contract Exists

Use one narrow scaffold first because:
- It increases deploy success rate.
- It keeps the current Happycapy scaffold reusable.
- It reduces prompt variance.
- It keeps the agent from choosing a stack that cannot run on Workers.
- It gives Happycapy a stable baseline for later backend-managed platform deployment.
