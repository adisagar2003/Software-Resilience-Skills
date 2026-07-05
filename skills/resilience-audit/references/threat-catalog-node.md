# Threat catalog — Node / Next.js

Use alongside the generic catalog. Each row: only include it if the
trigger is present in the repo.

| Threat | CIA | Trigger (what to look for) | Typical Recognition | Typical Resistance |
|---|---|---|---|---|
| npm supply-chain compromise (postinstall scripts) | I | `package.json` with lifecycle scripts, lockfile | CI audit step, lockfile diffs in review | pinned versions, `ignore-scripts` where viable, review major bumps |
| JWT/session secret leak or weak fallback | C/I | `jsonwebtoken`/`express-session` imports, `JWT_SECRET` env name | impossible-usage-pattern alerts on authed routes | secret only in prod env store; refuse to boot without it; rotate to invalidate sessions |
| API routes exposed without auth middleware | C | `app/api/` or `pages/api/` routes, Express routers lacking auth middleware | 401/403 vs 200 ratio monitoring, access logs | shared `requireAuth` middleware applied by default, deny-by-default routing |
| Serverless timeout / cold-start unavailability | A | `vercel.json`, netlify.toml, serverless function entrypoints | p95 latency + timeout-rate alerts | function timeouts sized to workload, streaming responses, keep bundles small |
| ORM migration-on-deploy data damage | I/A | `prisma migrate deploy` (or similar) in build/deploy commands | post-deploy health check counting core tables | reviewed additive migrations, restore point before deploy, rollback path |
| Client-side env leakage | C | `NEXT_PUBLIC_`/`VITE_` prefix on sensitive-looking names | bundle scanning for key patterns | server-only keys never prefixed; proxy provider calls through the API |
| Webhook forgery (billing/integrations) | I | stripe/polar/webhook route handlers | log every event with signature-verification result; alert on failures | verify provider signature on every event before acting |
| Paid-API spend abuse | A | server-side LLM/API clients on user-triggerable endpoints | provider spend alerts, per-user usage logging | auth + rate limits on the endpoint, hard spend cap at the provider |
