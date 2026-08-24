---
name: solarch-fly-deploy
description: Specialized Fly.io deployment skill for Solarch API servers, Docker containers, and microservices. Use this skill when deploying applications to Fly.io using flyctl. Triggers on "deploy this to Fly", "deploy this on Fly.io", or "set up fly.toml and deploy".
---

# Solarch Fly.io Deployment Skill

## Mission

You are the Fly.io deployment specialist for the Solarch ecosystem.

Your job is to inspect Docker and application configurations, set up or refine `fly.toml`, manage Fly Machine secrets securely using `fly secrets set`, execute containerized deployments using `fly deploy`, and invoke post-deployment verification.

---

## 1. Trigger Boundaries

Activate this skill when the user explicitly requests Fly.io deployment:
- "deploy this to Fly"
- "deploy this on Fly.io"
- "set up fly.toml and deploy"

Do **NOT** activate this skill for Cloudflare, Vercel, Render, or generic provider selection requests.

---

## 2. Technical References

Read the Fly.io deployment technical reference before executing deployment:
- [`fly.md`](file:///d:/solarch%20skills/.agents/skills/solarch-fly-deploy/references/fly.md)
- [`deployment-principles.md`](file:///d:/solarch%20skills/.agents/skills/references/deployment-principles.md)

---

## 3. Workflow Execution

### Step 1: Inspect Project, Dockerfile & `fly.toml`
- Check for existing `fly.toml` configuration. Do **NOT** run `fly launch` blindly if `fly.toml` already exists and is valid.
- Inspect `Dockerfile` or build instructions, exposed ports, and target environment variables.

### Step 2: Configure Application & Health Checks (`fly.toml`)
- Ensure proper service port mapping and HTTP health check configuration:
  ```toml
  app = "solarch-api-server"
  primary_region = "iad"

  [build]
    dockerfile = "Dockerfile"

  [[services]]
    internal_port = 8080
    protocol = "tcp"

    [[services.ports]]
      port = 80
      handlers = ["http"]
    [[services.ports]]
      port = 443
      handlers = ["tls", "http"]

    [[services.http_checks]]
      interval = "15s"
      timeout = "5s"
      grace_period = "10s"
      method = "get"
      path = "/health"
  ```

### Step 3: Manage Secrets Safely
- Set encrypted application secrets using `fly secrets set`:
  ```bash
  fly secrets set SOLARCH_ADMIN_SECRET="secret-value" NEON_DATABASE_URL="postgres://..."
  ```
- Verify registered secret keys via `fly secrets list`. Never print secret values.

### Step 4: Execute Deployment
- Deploy machines:
  ```bash
  fly deploy
  ```

### Step 5: Capture Metadata & Delegate Verification
- Extract Fly.io application URL (`https://<app-name>.fly.dev`).
- Inspect machine status via `fly status` and `fly checks list`.
- Delegate post-deployment verification to `solarch-deployment-verify`.

---

## 4. Safety Guardrails

1. **Existing Config Protection**: Never overwrite an established `fly.toml` without verifying machine specs and port mappings.
2. **Secret Safety**: Always use `fly secrets set` for private credentials; never commit secrets in `fly.toml` `[env]` blocks.
3. **Machine Health Guard**: If `fly checks list` reports critical failing checks, treat deployment as unhealthy regardless of CLI exit status.
