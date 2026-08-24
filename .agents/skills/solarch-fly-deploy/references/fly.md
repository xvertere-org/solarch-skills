# Fly.io Deployment Technical Reference

## 1. Fly.io Deployment Ecosystem

Fly.io runs applications globally on lightweight Fly Machines using Docker containers.

### Key Tools & Files

- **CLI Tool**: `flyctl` or `fly`.
- **Configuration File**: `fly.toml`.
- **Container Build**: `Dockerfile` or automatic buildpacks.

---

## 2. Configuration (`fly.toml`)

```toml
app = "solarch-api-server"
primary_region = "iad"

[build]
  dockerfile = "Dockerfile"

[env]
  PORT = "8080"
  SOLARCH_LOG_LEVEL = "info"

[[services]]
  protocol = "tcp"
  internal_port = 8080

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

---

## 3. Essential Commands

### Launching / Initializing Project Configuration
```bash
# Initializes fly.toml without auto-deploying if already present
fly launch --no-deploy
```

### Managing Environment Secrets
```bash
# Set encrypted secrets (triggers machine restart/deploy automatically)
fly secrets set SOLARCH_ADMIN_SECRET="secret-value" NEON_DATABASE_URL="postgres://..."

# List active secrets (keys and digests only)
fly secrets list
```

### Executing Deployment
```bash
# Build container and deploy machines
fly deploy
```

### Checking Application Status & Logs
```bash
# View app and machine health status
fly status

# View health check states specifically
fly checks list

# Stream realtime logs
fly logs
```

---

## 4. Rollback & Recovery
```bash
# List release history
fly releases

# Rollback to previous release image/version
fly releases deploy [RELEASE_VERSION]
```
