# Render Deployment Technical Reference

## 1. Render Deployment Ecosystem

Render supports automated Web Services, Static Sites, Background Workers, and managed PostgreSQL databases, defined either manually or through `render.yaml` infrastructure-as-code Blueprints.

### Key Concepts & Services

- **Web Services**: Persistent Node.js/Docker processes accepting HTTP requests.
- **Static Sites**: CDN-cached frontend distributions.
- **Blueprints**: `render.yaml` manifest specifying service architecture and environment configs.
- **Health Check Path**: HTTP probe endpoint configured via `healthCheckPath` (e.g. `/health`).

---

## 2. Blueprint Configuration (`render.yaml`)

```yaml
services:
  - type: web
    name: solarch-server-api
    env: docker # or node
    plan: starter
    healthCheckPath: /health
    envVars:
      - key: PORT
        value: 10000
      - key: SOLARCH_ENV
        value: production
      - key: DATABASE_URL
        sync: false # Managed secret set via Render Dashboard
```

---

## 3. Deployment & Secret Safety

### Environment Variables & Secrets
- Static values defined via `value`.
- Auto-generated passwords via `generateValue: true`.
- Sensitive secrets kept out of source control using `sync: false` and set via Dashboard or Secret Files (`/etc/secrets/<filename>`).

### Health Verification & Zero-Downtime
- Render polls `healthCheckPath` on port 10000 (or specified `PORT`).
- Successful HTTP 2xx/3xx response completes zero-downtime deployment.
- HTTP 4xx/5xx responses mark instance unhealthy and prevent traffic switching.

---

## 4. Deploy Triggering & Logs
- **Git-backed deploy**: Triggered automatically on push to designated branch or via Render Deploy Hook Webhook URL.
- **CLI / API Deploy**: Trigger build via Render REST API or Dashboard.
- **Status Inspection**: Monitor deployment events and logs via Render Dashboard or API.
