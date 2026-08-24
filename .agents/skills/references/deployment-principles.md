# Solarch Deployment Principles & Target Selection

## 1. Deployment Philosophy

In the Solarch ecosystem, a successful deployment requires full operational readiness and runtime health across the application and Solarch backend services.

> **CLI exit code `0` means deployment was accepted by the provider — NOT that the application is healthy.**

### Core Deployment Rules

1. **Verification-Driven**: A deployment is only complete when `solarch-deployment-verify` confirms network accessibility, application shell rendering, and Solarch API reachability.
2. **Environment & Target Isolation**: Treat `preview` and `production` as non-overlapping environments. Never execute production deployment commands on local/experimental branches without explicit authorization.
3. **Secret Protection**: Secrets (database connection strings, API tokens, signing keys) must never be committed to source control, echoed in terminal logs, or embedded in client-side bundles. Missing secrets must be reported directly to the user.
4. **Preserve Existing Configuration**: Inspect and preserve existing `wrangler.jsonc`/`wrangler.toml`, `vercel.json`, `render.yaml`, `fly.toml`, or `Dockerfile` configurations unless modification is strictly required.
5. **Rollback Awareness**: If verification fails after a production deployment, immediately identify the previous stable deployment version and present the safest non-destructive rollback steps.

---

## 2. Deployment Target Detection Matrix

Before selecting a deployment provider or executing build scripts, inspect the project structure, package manifests, and configuration files to determine the application archetype:

| Project Archetype | Indicators / Characteristics | Preferred Providers |
| :--- | :--- | :--- |
| **Static Frontend / SPA** | `vite`, `create-react-app`, HTML/JS static output (`dist/`, `build/`), no custom server logic. | Vercel, Cloudflare Pages/Workers Static Assets, Render Static Sites |
| **SSR / Full-Stack App** | Next.js, Remix, SvelteKit, Nuxt, Node.js SSR framework. | Vercel, Cloudflare Workers, Render Web Services, Fly.io |
| **Worker / Serverless API** | Cloudflare Workers, Hono, edge runtime handlers, `wrangler.jsonc`. | Cloudflare Workers |
| **Node.js API / Server** | Solarch Server, Express, Fastify, NestJS, SQLite/PostgreSQL driver logic, persistent process. | Render Web Services, Fly.io, Container runtime |
| **Containerized App** | `Dockerfile`, `docker-compose.yml`, custom OS dependencies or binary DB bindings. | Fly.io, Render Web Services |
| **Core Package / SDK** | `@solarch/core-client`, UI SDKs, shared libraries (no server/app entry point). | **Do NOT deploy directly** (Use `solarch-build-release` to publish npm packages) |

---

## 3. Standard Solarch Deployment Lifecycle

All deployment executions must adhere to the 13-stage lifecycle:

```text
1. Inspect (Framework, runtime, package scripts, Git status, existing provider configs)
        ↓
2. Determine Deployment Target (Static SPA, SSR, API, Worker, Container, or Library)
        ↓
3. Check Git & Project State (Clean worktree, active branch, commit hash)
        ↓
4. Inspect Environment Configuration (Public vars vs. Server-only secrets vs. Build-time vars)
        ↓
5. Validate Dependencies (Lockfile integrity, ESM/CJS compatibility)
        ↓
6. Run Validation Tests (Proportional test suite via CI/CD and testing skills)
        ↓
7. Execute Build (`npm run build` or framework-specific build pipeline)
        ↓
8. Configure Provider Deployment (Target configuration, environment mapping)
        ↓
9. Authenticate Provider (Token/session verification, non-interactive flags)
        ↓
10. Execute Deployment (Trigger preview or production deployment)
        ↓
11. Capture Deployment Metadata (URL, Deployment ID, commit hash, timestamp)
        ↓
12. Verify Deployment (Invoke `solarch-deployment-verify` against live URL)
        ↓
13. Report Result (Structured summary with health checklist and rollback status)
```

---

## 4. Environment Variable Classification & Safety

Always categorize environment variables prior to deployment:

- **Public Client Variables**: Embedded in frontend JS bundles at build time (e.g., `NEXT_PUBLIC_SOLARCH_URL`, `VITE_SOLARCH_APP_ID`). Must not contain sensitive secrets.
- **Server-Only Variables**: Injected at runtime for API endpoints (e.g., `PORT`, `SOLARCH_DB_DRIVER`, `LOG_LEVEL`).
- **Secrets & Credentials**: Private keys, database connection strings, master credentials (e.g., `SOLARCH_ADMIN_SECRET`, `NEON_DATABASE_URL`, `CLOUDFLARE_API_TOKEN`). Must be set via provider vault/secrets commands (`fly secrets set`, `wrangler secret put`, Vercel Dashboard / CLI env add, Render Dashboard).
- **Build-Time Variables**: Required during static asset generation or SSR bundling.

> [!CAUTION]
> Never print full secret values in console outputs or markdown reports. Display only variable names and presence status (e.g., `SOLARCH_ADMIN_SECRET: [SET]`).

---

## 5. Deployment Failure Boundaries

When diagnosing a failed deployment or verification failure, classify the root failure boundary:

1. **Build Failure**: Syntax errors, missing modules, TypeScript compilation errors during `npm run build`.
2. **Provider Configuration Failure**: Invalid `wrangler.jsonc`, `vercel.json`, `render.yaml`, or `fly.toml` syntax or unsupported routing rules.
3. **Authentication / Quota Failure**: Expired provider CLI token, missing permissions, or resource limit reached.
4. **Runtime Crash**: Process exits on boot due to missing environment variables, DB connection failure, or unhandled exception.
5. **Health Check Failure**: Service accepts HTTP connections but fails `/health` probe (returns 5xx or times out).
6. **Solarch API Boundary Failure**: App shell loads but REST/WebSocket calls to Solarch Server fail (CORS mismatch, unauthorized, incorrect endpoint URL).
