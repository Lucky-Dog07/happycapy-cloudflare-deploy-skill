# Security And Rationale

Read this file when you need to explain why the skill works this way.

## Why Ask Only For Account ID And API Token

Ask only for `CLOUDFLARE_ACCOUNT_ID` and `CLOUDFLARE_API_TOKEN` because:
- Those two values are sufficient for the agent to create or update the required Workers for Platforms resources.
- It keeps the user interaction short and predictable.
- It avoids pushing Cloudflare naming and platform setup details onto the user.

The agent should derive the rest:
- Dispatch Namespace name
- Dispatch Worker name
- User Worker name

## Why Avoid Persisting The Token

Do not persist the token because:
- A Cloudflare API token can deploy code on the user's account.
- Prompt-provided tokens are already sensitive; persisting them increases blast radius.
- The eventual product direction is backend-managed secrets, so avoid building bad habits into the project files.

## Why Use Workers for Platforms Now

Use Workers for Platforms now because:
- Happycapy ultimately needs a platform-managed deployment model.
- A Dispatch Worker plus User Worker model matches that direction better than a single regular Worker.
- The app can still be exposed through a stable public URL while keeping the app code isolated as a User Worker.

## Why Return The Dispatcher URL

Return the Dispatcher URL because:
- User Workers do not get preview URLs.
- The Dispatch Worker is the public entrypoint that Cloudflare exposes on `workers.dev`.
- A path-based Dispatcher URL is enough to prove the deployment survives sandbox shutdown.

## Why Preserve The Existing Scaffold

Use one scaffold because:
- Happycapy needs predictable agent output.
- Cloudflare Workers has runtime constraints that many common Node stacks violate.
- The current Vite + React + Tailwind + Hono + Wrangler scaffold is already suitable as the app side of the deployment.
- A fixed scaffold makes later platform-managed deploy automation easier.

## What Changes In Phase Two

Move to backend-managed secrets and platform ownership when:
- Happycapy needs safer token handling
- Happycapy wants one-click deploy without pasting credentials
- Happycapy wants tenant isolation, domain automation, deploy records, and lifecycle management
