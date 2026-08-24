---
name: solarch-vercel-deploy
description: Specialized Vercel deployment skill for Solarch web applications, Next.js, and static/SSR frontends. Use this skill when deploying projects to Vercel infrastructure using Vercel CLI or CI workflows. Triggers on "deploy this to Vercel", "deploy the frontend to Vercel", "deploy through GitHub Actions to Vercel", or "use Vercel for this app".
---

# Solarch Vercel Deployment Skill

## Mission

You are the Vercel deployment specialist for the Solarch ecosystem.

Your job is to execute Vercel deployments using the Vercel CLI, manage Preview vs Production environments, safely configure environment variables, and delegate post-deployment verification.

---

## 1. Trigger Boundaries

Activate this skill when the user explicitly requests Vercel deployment:
- "deploy this to Vercel"
- "deploy the frontend to Vercel"
- "use Vercel for this app"

Do **NOT** activate this skill for Cloudflare, Render, Fly.io, or generic target selection requests.

---

## 2. Technical References

Read the Vercel deployment technical reference before executing deployment:
- [`vercel.md`](file:///d:/solarch%20skills/.agents/skills/solarch-vercel-deploy/references/vercel.md)
- [`deployment-principles.md`](file:///d:/solarch%20skills/.agents/skills/references/deployment-principles.md)

---

## 3. Workflow Execution

### Step 1: Inspect Framework & Vercel Linking
- Check project framework (Next.js, Vite, Remix, Svelte, static HTML).
- Verify Vercel project linking via `.vercel/project.json` or run:
  ```bash
  npx vercel link --yes
  ```

### Step 2: Configure Environment Variables & Settings
- Synchronize environment settings:
  - For Preview: `npx vercel pull --yes --environment=preview`
  - For Production: `npx vercel pull --yes --environment=production`
- Verify required public vars (e.g. `NEXT_PUBLIC_SOLARCH_URL`) and server secrets without logging values.

### Step 3: Build & Deploy
- **Preview Deployment**:
  ```bash
  npx vercel
  ```
- **Production Deployment**:
  ```bash
  npx vercel --prod
  ```
- **Prebuilt Pipeline (where applicable)**:
  ```bash
  npx vercel pull --yes --environment=production
  npx vercel build --prod
  npx vercel deploy --prebuilt --prod
  ```

### Step 4: Capture Metadata & Verify
- Extract deployment URL from Vercel CLI output.
- Invoke `solarch-deployment-verify` to validate live HTTP accessibility, frontend rendering, and Solarch API reachability.

---

## 4. Safety Guardrails

1. **Preview vs Production Isolation**: Default to preview deployments unless `--prod` or explicit production deployment intent is established.
2. **Secret Safety**: Never commit `.env.production.local` containing secrets or echo environment tokens into output markdown.
3. **Prebuilt Warning**: If using `--prebuilt`, ensure environment variables were pulled prior to building, as system env vars are not available during prebuilt uploads.
