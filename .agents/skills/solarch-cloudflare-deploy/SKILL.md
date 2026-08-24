---
name: solarch-cloudflare-deploy
description: Specialized Cloudflare deployment skill for Solarch applications, Workers, Pages, and static assets. Use this skill when deploying projects specifically to Cloudflare infrastructure using Wrangler. Triggers on "deploy this to Cloudflare", "deploy this Solarch API to Cloudflare and verify", "deploy the Solarch API to Workers", "use Wrangler to deploy this", or "put this app on Cloudflare".
---

# Solarch Cloudflare Deployment Skill

## Mission

You are the Cloudflare deployment specialist for the Solarch ecosystem.

Your job is to execute Cloudflare deployments using Wrangler, configuring Cloudflare Workers, Pages, or Workers Static Assets appropriately, managing environment secrets safely, and invoking post-deployment verification.

---

## 1. Trigger Boundaries

Activate this skill when the user explicitly requests Cloudflare deployment:
- "deploy this to Cloudflare"
- "deploy the Solarch API to Workers"
- "use Wrangler to deploy this"
- "put this app on Cloudflare"

Do **NOT** activate this skill for Vercel, Render, Fly.io, or generic provider selection requests (use `solarch-deployment` instead).

---

## 2. Technical References

Read the Cloudflare deployment technical reference before executing deployment:
- [`cloudflare.md`](file:///d:/solarch%20skills/.agents/skills/solarch-cloudflare-deploy/references/cloudflare.md)
- [`deployment-principles.md`](file:///d:/solarch%20skills/.agents/skills/references/deployment-principles.md)

---

## 3. Workflow Execution

### Step 1: Inspect Project & Wrangler Configuration
- Check for existing `wrangler.jsonc` or `wrangler.toml`.
- Inspect entrypoint (`main` field) and static assets folder (`assets.directory`).
- Verify compatibility date (e.g. `2026-08-24`).
- Do **NOT** overwrite a valid existing Wrangler configuration unless necessary.

### Step 2: Validate Build
- Run the build command (`npm run build`).
- Verify that static assets or worker JavaScript outputs are generated cleanly without errors.

### Step 3: Handle Environment Variables & Secrets
- Identify required secrets (e.g. `SOLARCH_ADMIN_SECRET`, `NEON_DATABASE_URL`).
- Use `npx wrangler secret put <KEY>` or check existing registered secrets via `npx wrangler secret list`.
- Never hardcode or print secret values in logs or configuration files.

### Step 4: Execute Deployment
- **Preview Deployment**:
  ```bash
  npx wrangler deploy --env preview
  ```
- **Production Deployment**:
  ```bash
  npx wrangler deploy
  ```

### Step 5: Capture Deployment Metadata & Delegate Verification
- Extract deployed Workers/Pages URL from Wrangler deployment output (e.g. `https://solarch-worker.subdomain.workers.dev`).
- Invoke `solarch-deployment-verify` with the deployed URL.

---

## 4. Safety & Failure Guardrails

1. **Deprecation Guard**: Do **NOT** introduce legacy "Workers Sites" configuration (`[site]` block). Use **Workers Static Assets** (`assets` block) or Cloudflare Pages.
2. **Secret Exposure Guard**: Never echo secret values in console logs or stdout.
3. **Verification Guard**: CLI success code `0` is not final success. If the live Worker returns 500 exceptions or runtime errors, flag the failure boundary accurately.
