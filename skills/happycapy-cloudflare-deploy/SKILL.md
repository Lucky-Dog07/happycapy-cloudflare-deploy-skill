---
name: happycapy-cloudflare-deploy
description: Deploy a Happycapy full-stack app to Cloudflare Workers for Platforms using only a user-provided Cloudflare Account ID and API token, automatically normalize the repo into a Workers-for-Platforms-deployable shape, preserve the existing Happycapy Vite + React + Tailwind + Hono + Wrangler scaffold when possible, and return the final public platform URL. Use when the user asks to deploy, obtain a persistent Cloudflare URL, move a Happycapy app onto Workers for Platforms, or keep a Happycapy app online after the sandbox closes. This skill also covers Chinese requests such as “部署到 Cloudflare”, “用 Workers for Platforms 部署”, “生成长期可访问 URL”, or “沙盒关掉后还能访问”.
---

# Happycapy Cloudflare Deploy

## Purpose

Use this skill to turn a Happycapy project into a Cloudflare-hosted deployment that remains reachable after the Happycapy sandbox stops.

Treat this as a Workers for Platforms deploy path:
- Ask the user only for `CLOUDFLARE_ACCOUNT_ID` and `CLOUDFLARE_API_TOKEN`.
- Deploy the app as a User Worker behind a generated Dispatch Worker.
- Return the Dispatch Worker URL plus the User Worker path as the public entry URL.
- Do not treat this as the final production security model.

## Workflow

1. Inspect the project before changing anything, then normalize automatically.
- Read `package.json`, build scripts, framework config, and entry files.
- Decide whether the app already fits the Happycapy Cloudflare scaffold.
- If it is reasonably convertible, perform the normalization without asking the user for extra approval.
- Preserve working app code and deploy scripts when possible; add the minimum Workers for Platforms wrapper files needed.
- Stop only when the repo fundamentally depends on unsupported runtime assumptions. Use [`references/scaffold-contract.md`](./references/scaffold-contract.md).

2. Keep secrets out of the repository.
- Ask the user for `CLOUDFLARE_ACCOUNT_ID` and `CLOUDFLARE_API_TOKEN` only when deployment is imminent.
- Do not ask for namespace names, dispatcher names, script names, custom domains, or other Cloudflare identifiers unless the user explicitly wants to override defaults.
- Keep the token only in the current command environment.
- Never write the token to source files, `.env`, `wrangler.toml`, `wrangler.jsonc`, shell rc files, logs, or docs.
- Prefer a single command invocation with inline environment variables over a persistent export.

3. Normalize the project to the Workers for Platforms deploy shape.
- Ensure the client can build to static assets.
- Ensure API routes run in a Worker-compatible entrypoint.
- Prefer React + Vite for UI, Tailwind for styling, Hono for API routing, and Wrangler for deploy.
- Derive deterministic resource names from the repo slug and create or reuse the Dispatch Namespace and Dispatch Worker automatically.
- Preserve the existing Happycapy deploy scaffold when it already works; layer Workers for Platforms on top instead of rebuilding the app from scratch.
- Do not configure a custom domain unless the user explicitly asks.
- Use the guidance in [`references/deploy-workflow.md`](./references/deploy-workflow.md).

4. Deploy only after a successful build.
- Run dependency install if needed.
- Run the production build first and fix build/runtime issues before deploy.
- Deploy the Dispatch Worker with inline Cloudflare credentials.
- Deploy the User Worker into the Dispatch Namespace with inline Cloudflare credentials.
- Capture the resulting platform URL and report it back.

5. Report the result in a structured way.
- State whether the deploy succeeded.
- Give the public URL.
- List changed files.
- State any follow-up needed for later platform-managed deployment.

## Required Rules

- Do not promise that this token flow is secure enough for general availability.
- Do not persist Cloudflare credentials anywhere in the project.
- Do not silently change to another hosting provider.
- Do not introduce Node-only server dependencies that are incompatible with Workers.
- Do not treat the sandbox preview URL as the final deployment URL.
- Do not make the user hand-create Workers for Platforms resources that the agent can create automatically.
- Do not ask the user for anything beyond `CLOUDFLARE_ACCOUNT_ID` and `CLOUDFLARE_API_TOKEN` during the normal flow.

## Decision Rules

- If the app is a simple SPA, configure SPA asset fallback.
- If the app has `/api/*` routes, configure Worker-first routing for those routes.
- If the app depends on unsupported native Node APIs or long-lived processes, say so clearly and stop or migrate the implementation.
- If the repo already has a valid Wrangler-based app Worker, keep it as the User Worker target and add only the Dispatcher layer.
- If Workers for Platforms is not enabled on the account, namespace creation is blocked, or the token lacks permission, surface that blocker clearly and stop.
- If the user asks for “stable preview” or “长期可访问链接”, explain that User Workers do not get preview URLs and return the Dispatcher URL instead.
- If the user asks for platform-scale multi-tenant hosting, use [`references/security-and-rationale.md`](./references/security-and-rationale.md).

## Outputs

Return all of the following:
- Deployment status.
- Public URL.
- Derived resource names.
- Commands run.
- Files created or changed.
- Risks or limitations that still remain.

## References

- Scaffold contract: [`references/scaffold-contract.md`](./references/scaffold-contract.md)
- Deploy procedure: [`references/deploy-workflow.md`](./references/deploy-workflow.md)
- Security and product rationale: [`references/security-and-rationale.md`](./references/security-and-rationale.md)

## Database-Backed Variant

When the Happycapy app already includes a database schema, keep the existing deploy path and layer persistence on top instead of replacing the whole architecture.

Use these extra rules:
- Preserve the existing business tables and API shape when they are already coherent.
- Prefer Cloudflare D1 as the persistence target for data that must survive Worker restarts.
- Treat local SQLite, JSON seed data, or in-memory mock storage as migration inputs, not as the final production persistence layer.
- Add the minimum required D1 pieces such as schema files, seed files, Worker queries, and Wrangler D1 bindings.
- Keep returning the same style of public Dispatcher URL after deployment; the database upgrade should not change the external access pattern.

Typical examples include:
- carts that must survive refreshes and cold starts
- orders that must remain queryable after deployment
- user addresses and profile data that need durable writes
