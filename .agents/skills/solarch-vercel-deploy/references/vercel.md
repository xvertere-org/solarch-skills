# Vercel Deployment Technical Reference

## 1. Vercel Deployment Ecosystem

Vercel provides native deployment integration for Next.js, React, Vite, Svelte, and static/SSR web applications with preview branch deployments and production promotion workflows.

### Key Concepts & Tooling

- **CLI Tool**: `npx vercel` or `vercel`.
- **Project Configuration**: `vercel.json` (optional override).
- **Environment Isolation**: Distinct **Preview** vs **Production** environment variable scopes.
- **Local Building**: `vercel pull`, `vercel build`, and `vercel deploy --prebuilt`.

---

## 2. Environment Variables & Linking Workflow

### Linking Local Directory to Vercel Project
```bash
# Link project non-interactively or interactively
npx vercel link --yes
```

### Pulling Settings & Environment Variables
```bash
# Pull preview environment settings into .vercel/
npx vercel pull --environment=preview

# Pull production environment settings into .vercel/
npx vercel pull --environment=production
```

### Managing Secrets & Env Vars via CLI
```bash
# Add environment variable safely
npx vercel env add SOLARCH_API_URL production

# List environment variables (names only)
npx vercel env ls
```

---

## 3. Deployment Commands

### Preview Deployment (Default)
```bash
# Triggers preview deployment and outputs unique preview URL
npx vercel
```

### Production Deployment (`--prod`)
```bash
# Promotes build to live production URL
npx vercel --prod
```

### Prebuilt Deployment Pipeline (CI / Custom Build)
```bash
# 1. Pull environment configuration
npx vercel pull --yes --environment=production

# 2. Build local output bundle
npx vercel build --prod

# 3. Deploy prebuilt bundle directly
npx vercel deploy --prebuilt --prod
```

---

## 4. Rollback & Inspection
```bash
# Inspect active project deployments
npx vercel ls

# Inspect specific deployment status & details
npx vercel inspect [DEPLOYMENT_URL_OR_ID]

# Alias / Rollback production domain to previous deployment
npx vercel alias set [PREVIOUS_DEPLOYMENT_URL] [PRODUCTION_DOMAIN]
```
