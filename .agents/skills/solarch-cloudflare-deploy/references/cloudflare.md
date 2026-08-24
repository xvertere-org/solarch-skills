# Cloudflare Deployment Technical Reference

## 1. Cloudflare Workers & Static Assets Ecosystem

Cloudflare's modern deployment model unifies full-stack applications, Workers edge logic, and static assets into a single deployment pipeline.

> **Current Recommendation**: Prefer **Workers Static Assets** for full-stack apps and modern web frameworks over legacy Workers Sites (deprecated in Wrangler v4).

### Key Concepts & CLI Tooling

- **CLI Tool**: `npx wrangler` (Wrangler v3/v4).
- **Configuration File**: `wrangler.jsonc` (recommended) or `wrangler.toml`.
- **Static Assets Binding**: `assets.directory` points to static build output (e.g. `./dist`, `./out`). Access via `env.ASSETS` or automatic asset serving.

---

## 2. Configuration Patterns

### `wrangler.jsonc` Example (Full-Stack / Worker + Static Assets)
```jsonc
{
  "$schema": "./node_modules/wrangler/config-schema.json",
  "name": "solarch-worker-app",
  "main": "src/index.ts",
  "compatibility_date": "2026-08-24",
  "assets": {
    "directory": "./dist",
    "binding": "ASSETS"
  },
  "vars": {
    "ENVIRONMENT": "production"
  }
}
```

---

## 3. Essential Deployment Commands

### Authentication Check
```bash
npx wrangler whoami
```

### Preview / Staging Deployment
```bash
# Deploys to a preview environment or preview URL
npx wrangler deploy --env preview
```

### Production Deployment
```bash
# Deploys to live production environment
npx wrangler deploy
```

### Managing Secrets & Environment Variables
```bash
# Upload a secret safely without printing or committing it
npx wrangler secret put SOLARCH_ADMIN_SECRET

# List registered secrets (names only, values hidden)
npx wrangler secret list
```

### Viewing Realtime Deployment Logs
```bash
npx wrangler tail
```

---

## 4. Rollback & Versioning
```bash
# List previous deployment versions
npx wrangler deployments list

# Rollback to a specific deployment ID
npx wrangler rollback [DEPLOYMENT_ID]
```
