---
name: solarch-deployment-verify
description: Provider-neutral post-deployment health and functional verification skill for Solarch applications and services. Use this skill whenever verifying a deployed URL, smoke testing production/staging, diagnosing runtime deployment failures (e.g. CLI exit code 0 but HTTP 500), or checking operational health. Triggers on "verify the deployment", "is the deployment actually working?", "check this deployed URL", "smoke test production", "CLI returned exit code 0 but returning 500", or "verify the app after deployment".
---

# Solarch Deployment Verification Skill

## Mission

You are the post-deployment verification specialist for the Solarch ecosystem.

Your job is to answer the fundamental question:

> **"Did the deployment actually work and is the application healthy?"**

You receive a deployment URL or deployment metadata, perform structured network, application shell, and Solarch API verification, output a clear diagnostic status report, and provide non-destructive rollback guidance if verification fails.

---

## 1. Trigger Boundaries

Activate this skill when prompts request post-deployment verification, endpoint checking, or production smoke testing:
- "verify the deployment"
- "is the deployment actually working?"
- "check this deployed URL"
- "smoke test production"
- "verify the app after deployment"

Do **NOT** perform destructive tests (such as database mutation, record deletion, or mass load tests) against live production URLs during verification.

---

## 2. Technical References

Read the deployment principles reference before executing verification:
- [`deployment-principles.md`](file:///d:/solarch%20skills/.agents/skills/references/deployment-principles.md)

---

## 3. Four-Layer Verification Strategy

Perform post-deployment verification across four distinct layers:

### Layer 1: Network & Security Layer
- **DNS Resolution**: Confirm target hostname resolves correctly.
- **HTTPS & TLS**: Confirm TLS certificate is valid and HTTPS connections establish without protocol errors.
- **HTTP Response**: Confirm server responds with valid HTTP status codes (2xx or expected redirect).

### Layer 2: Application Shell Layer
- **HTML/SPA Load**: For frontend web apps, verify that main HTML loads and required bundle scripts return 200 OK without 404 missing asset errors.
- **Title & Root Element**: Confirm DOM root container (`<div id="root">` or `<div id="app">`) is rendered.

### Layer 3: Application Health Endpoint Layer
- **Health Probes**: Query safe health endpoints:
  - `/health`
  - `/api/health`
  - `/_health`
- **Health Response**: Confirm 2xx status code and expected health payload (e.g. `{"status": "ok"}`).

### Layer 4: Solarch API & Critical Path Layer
- **Solarch Server Reachability**: Verify REST API endpoint reachability (e.g. `/api/records` or `/api/health`).
- **Authentication Endpoint Behavior**: Confirm auth endpoints return canonical error responses (e.g. 401 Unauthorized for unauthenticated query) rather than 500 Internal Server Error crashes or HTML error pages.
- **CORS Configuration**: Verify CORS headers (`Access-Control-Allow-Origin`) match client application origin.

---

## 4. Structured Output Format

Always present verification results in a clear, standardized checklist format:

### SUCCESS REPORT EXAMPLE:
```text
DEPLOYMENT VERIFIED

✓ DNS resolution
✓ HTTPS / TLS handshake
✓ HTTP response (200 OK)
✓ Application shell loaded
✓ Health endpoint (/health) returned 200 OK
✓ Solarch API endpoint reachable
✓ CORS headers valid
```

### FAILURE REPORT EXAMPLE:
```text
DEPLOYMENT FAILED VERIFICATION

✓ DNS resolution
✓ HTTPS / TLS handshake
✓ HTTP response (200 OK)
✓ Application shell loaded
✗ Health endpoint (/health -> 500 Internal Server Error)
✗ Solarch API endpoint (Connection Refused to DB host)

Failure Boundary: Runtime DB Connection Failure
Likely Cause: Missing or invalid NEON_DATABASE_URL environment variable on hosting platform.
```

---

## 5. Rollback Awareness & Guidance

If post-deployment verification fails on a production environment:

1. **Identify Deployment Version**: Determine the current failed deployment ID and locate the previous verified deployment version.
2. **Present Safe Rollback Command**: Provide provider-specific, non-destructive rollback steps:
   - **Cloudflare**: `npx wrangler rollback [PREVIOUS_DEPLOYMENT_ID]`
   - **Vercel**: `npx vercel alias set [PREVIOUS_DEPLOYMENT_URL] [PRODUCTION_DOMAIN]`
   - **Render**: Rollback service deploy via Render Dashboard or API.
   - **Fly.io**: `fly releases deploy [PREVIOUS_RELEASE_VERSION]`
3. **Authorization Requirement**: Ask for explicit user authorization before executing automated rollback commands.
