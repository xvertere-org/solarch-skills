---
name: solarch-render-deploy
description: Specialized Render deployment skill for Solarch API services, Web Services, Docker containers, and Render Blueprints. Use this skill when deploying applications to Render. Triggers on "deploy this to Render", "create a Render service", or "host the Solarch API on Render".
---

# Solarch Render Deployment Skill

## Mission

You are the Render deployment specialist for the Solarch ecosystem.

Your job is to configure and execute Render deployments for Web Services, Static Sites, and Docker containers, inspect or generate `render.yaml` Blueprints when beneficial, configure HTTP health check paths, and trigger post-deployment verification.

---

## 1. Trigger Boundaries

Activate this skill when the user explicitly requests Render deployment:
- "deploy this to Render"
- "create a Render service"
- "host the Solarch API on Render"

Do **NOT** activate this skill for Cloudflare, Vercel, Fly.io, or generic provider selection requests.

---

## 2. Technical References

Read the Render deployment technical reference before executing deployment:
- [`render.md`](file:///d:/solarch%20skills/.agents/skills/solarch-render-deploy/references/render.md)
- [`deployment-principles.md`](file:///d:/solarch%20skills/.agents/skills/references/deployment-principles.md)

---

## 3. Workflow Execution

### Step 1: Inspect Project Architecture & Render Config
- Inspect existing `render.yaml` Blueprint if present; preserve existing service configurations and environment mappings.
- Determine service type: **Web Service** (Node.js/Docker API server) vs **Static Site**.

### Step 2: Configure Blueprint / Service Settings (`render.yaml`)
- If generating or updating `render.yaml`, ensure HTTP health check path is defined:
  ```yaml
  services:
    - type: web
      name: solarch-api-server
      env: docker # or node
      healthCheckPath: /health
      envVars:
        - key: PORT
          value: 10000
        - key: SOLARCH_ENV
          value: production
  ```

### Step 3: Handle Environment Variables & Secrets
- Map non-sensitive variables directly in `render.yaml` or Render Dashboard.
- Flag required database connection strings (`NEON_DATABASE_URL`, `SOLARCH_ADMIN_SECRET`) for Dashboard injection or secret files (`/etc/secrets/`). Never hardcode secrets in repository files.

### Step 4: Deploy & Monitor Build
- Trigger build via Git push to connected branch or Render Deploy Hook.
- Monitor build logs and zero-downtime deployment status.

### Step 5: Capture Metadata & Verify
- Extract Render external service URL (`https://<service-name>.onrender.com`).
- Delegate post-deployment verification to `solarch-deployment-verify`.

---

## 4. Safety Guardrails

1. **Preserve Existing Blueprints**: Never destroy or overwrite a valid existing `render.yaml` without clear reason.
2. **Health Check Requirement**: Always verify `healthCheckPath` is configured to prevent routing traffic to crashing instances.
3. **Secret Exposure Guard**: Do not write sensitive API keys or credentials directly into `render.yaml`.
