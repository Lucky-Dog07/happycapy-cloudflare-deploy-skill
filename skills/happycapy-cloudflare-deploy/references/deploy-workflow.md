# Deploy Workflow

Use this workflow when the user wants a stable Cloudflare URL now and the skill should handle the Workers for Platforms shape automatically.

## 1. Inspect

Read:
- `package.json`
- `wrangler.jsonc` or `wrangler.toml`
- `vite.config.*`
- Worker entry files
- Frontend build output settings

Determine:
- Is the app already compatible with a standard Cloudflare Worker build?
- Can it be normalized into the Happycapy scaffold without a full rewrite?
- Are there `/api/*` routes that must execute before asset fallback?

## 2. Ask Only For Credentials

When deployment is imminent, ask the user for only:
- `CLOUDFLARE_ACCOUNT_ID`
- `CLOUDFLARE_API_TOKEN`

Do not ask for:
- Dispatch Namespace name
- Dispatch Worker name
- User Worker script name
- Custom domain

Derive those automatically from the repo slug.

Recommended naming pattern:
- Dispatch Namespace: `hc-<slug>-ns`
- Dispatch Worker: `hc-<slug>-dispatcher`
- User Worker: `hc-<slug>-app`

## 3. Normalize

Keep the existing app scaffold whenever possible.

Preferred normalization:
- Reuse the existing Worker entrypoint as the User Worker entrypoint.
- Keep the frontend build output in `dist/`.
- Keep SPA fallback enabled when needed.
- Keep `/api/*` on the User Worker when API routes exist.
- Add a dedicated Dispatch Worker entrypoint and Wrangler config if they do not already exist.

Cloudflare's current asset routing model supports:
- `assets.directory`
- `assets.not_found_handling = "single-page-application"` for SPA fallback
- `assets.binding = "ASSETS"` when the Worker needs asset access
- `assets.run_worker_first = ["/api/*"]` for API-first routing

Preferred Dispatch Worker behavior:
- Read the first URL path segment as the User Worker name.
- Remove that segment before forwarding the request.
- Call `env.DISPATCHER.get(workerName)` to route the request.
- Return a clear `404` when no worker name is present.

## 4. Prepare Workers for Platforms Resources

Create or reuse the Dispatch Namespace automatically.

Preferred namespace creation flow:
- Try the Cloudflare API to create the namespace.
- If the namespace already exists, continue.
- If the account does not have Workers for Platforms enabled, stop with a clear error.

When the repo lacks a dispatcher config, generate one that binds the remote namespace:

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

## 5. Build

Run the build before deployment.

Preferred command:

```bash
npm run build
```

Fix:
- Missing dependencies
- Worker runtime incompatibilities
- Wrong output directory
- Frontend route fallback issues

## 6. Deploy

Use inline environment variables only for deploy commands.

Preferred command pattern for the Dispatch Worker:

```bash
CLOUDFLARE_ACCOUNT_ID=... CLOUDFLARE_API_TOKEN=... npx wrangler deploy --config wrangler.dispatcher.jsonc
```

Preferred command pattern for the User Worker:

```bash
CLOUDFLARE_ACCOUNT_ID=... CLOUDFLARE_API_TOKEN=... npx wrangler deploy --config wrangler.user.jsonc --name hc-example-app --dispatch-namespace hc-example-ns
```

If the repo already defines a working app deploy config, keep it as the basis for the User Worker config instead of rewriting the app structure.

Do not:
- Commit credentials
- Save credentials to `.env` for this flow
- Add credentials to shell startup files

## 7. Return Result

Always return:
- The final platform URL
- The derived Dispatch Namespace, Dispatch Worker, and User Worker names
- The exact deploy command pattern used without echoing secrets
- Which files were added or edited
- Any remaining limitation

Default public URL shape:

```text
https://<dispatch-worker>.<subdomain>.workers.dev/<user-worker-name>/
```

User Workers do not get preview URLs, so validate and report the Dispatch Worker URL instead.

## 8. Failure Handling

If deployment fails:
- Surface the actual build, API, or Wrangler error
- Identify whether the root cause is scaffold mismatch, permissions, missing Workers for Platforms access, or runtime incompatibility
- Stop after the first clear blocker unless the fix is straightforward
