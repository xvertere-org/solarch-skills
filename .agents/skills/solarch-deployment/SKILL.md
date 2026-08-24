---
name: solarch-deployment
description: Solarch master deployment orchestrator. Use this skill whenever the user asks to deploy a Solarch application or project, determine deployment strategy, set up hosting, or prepare a project for production. Triggers on "deploy this Solarch app", "how should we deploy this project?", "deploy this application", "set up deployment", "prepare this project for deployment", or "deploy this to a hosting provider".
---

# Solarch Deployment Orchestrator

## Mission

You are the master deployment orchestrator for the Solarch ecosystem.

Your job is to inspect a Solarch project, determine its target archetype, evaluate environment readiness, select the appropriate hosting provider, execute the deployment workflow, and delegate provider-specific deployment and verification tasks to specialized skills.

> **Crucial Rule**: CLI exit code `0` means deployment was accepted by the provider — NOT that the application is healthy. Final deployment success requires post-deployment verification.

---

## 1. Trigger & Scope Boundaries

### Active Triggers
Activate this skill when prompts request generic deployment assistance, deployment strategy selection, or target detection:
- "deploy this Solarch app"
- "how should we deploy this project?"
- "deploy this application"
- "set up deployment"
- "prepare this project for deployment"
- "deploy this to a hosting provider"

### Subagent Delegation Boundary
- When an explicit provider is specified by the user (e.g., "Deploy to Vercel" or "Deploy to Cloudflare Workers"), immediately respect the selection unless there is a fatal technical incompatibility, and delegate execution to the specialized provider skill:
  - `solarch-cloudflare-deploy`
  - `solarch-vercel-deploy`
  - `solarch-render-deploy`
  - `solarch-fly-deploy`
- Once provider deployment completes, delegate post-deployment verification to `solarch-deployment-verify`.
- For publishing npm packages or core SDK artifacts, do **NOT** attempt cloud application deployment. Delegate to `solarch-build-release`.

---

## 2. Shared Principles & References

Before executing deployment orchestration, read the shared deployment reference:
- [`deployment-principles.md`](file:///d:/solarch%20skills/.agents/skills/references/deployment-principles.md)

Keep in mind Solarch architectural boundaries from:
- [`solarch-architecture.md`](file:///d:/solarch%20skills/.agents/skills/references/solarch-architecture.md)

---

## 3. Deployment Target Detection

Before taking any action, inspect the repository to categorize the project archetype:

1. **Static Frontend / SPA**:
   - Files/Configs: `index.html`, `vite.config.ts`, `dist/`, single-page routing without Node server.
   - Target Provider Options: **Vercel**, **Cloudflare Pages / Workers Static Assets**, **Render Static Sites**.

2. **SSR / Full-Stack Application**:
   - Files/Configs: `next.config.js`, `remix.config.js`, `svelte.config.js`, server-side routing logic.
   - Target Provider Options: **Vercel**, **Cloudflare Workers**, **Render Web Services**, **Fly.io**.

3. **Worker / Serverless Application**:
   - Files/Configs: `wrangler.jsonc`, `wrangler.toml`, Hono / Worker fetch handlers, edge runtime logic.
   - Target Provider Options: **Cloudflare Workers**.

4. **Node.js API / Server**:
   - Files/Configs: Solarch Server, Express/Fastify server scripts, database connections (SQLite/Postgres), background worker tasks.
   - Target Provider Options: **Render Web Services**, **Fly.io**.

5. **Containerized Application**:
   - Files/Configs: `Dockerfile`, `docker-compose.yml`, binary native dependencies.
   - Target Provider Options: **Fly.io**, **Render Web Services**.

6. **Package / Library (Non-Deployable directly)**:
   - Files/Configs: `@solarch/core-client`, client SDKs, modular component libraries without application entrypoints.
   - Action: **Halt deployment workflow**. Explain that package libraries are published to registries via `solarch-build-release`, not deployed to cloud hosting platforms.

---

## 4. 13-Stage Deployment Lifecycle

Follow this strict lifecycle sequence:

```text
1. Inspect (Framework, package.json scripts, build output, git status, existing provider configs)
        ↓
2. Determine Deployment Target (Classify target archetype)
        ↓
3. Check Git & Project State (Clean worktree, active branch, commit hash)
        ↓
4. Inspect Environment Configuration (Public vars vs. secrets; verify missing required vars)
        ↓
5. Validate Dependencies (Lockfile presence, module resolution)
        ↓
6. Run Validation Tests (Execute unit/contract tests via solarch-testing / solarch-ci-cd)
        ↓
7. Execute Build (`npm run build` or framework build step)
        ↓
8. Configure Provider Deployment (Target configuration & environment mapping)
        ↓
9. Authenticate Provider (Verify CLI session / tokens)
        ↓
10. Execute Deployment (Trigger preview vs. production deployment)
        ↓
11. Capture Deployment Metadata (Deployment URL, Deployment ID, commit hash)
        ↓
12. Verify Deployment (Delegate to solarch-deployment-verify)
        ↓
13. Report Result (Present comprehensive status report)
```

---

## 5. Preview vs. Production Safety Rules

Always distinguish between **Preview** and **Production** deployment intent:

- **Preview Deployment**:
  - Used for feature branches, pull requests, or local experiment testing.
  - Generates isolated preview URL without touching live user traffic or production databases.

- **Production Deployment**:
  - Used strictly for main/master branches or explicitly authorized production releases.
  - Requires pre-deployment verification: clean git working tree, verified build status, presence of required production environment secrets.

> [!WARNING]
> Never execute a production deployment on an experimental or uncommitted local branch without explicit user confirmation.

---

## 6. Environment & Secret Safety

- **Public vs. Secret Variables**: Distinguish frontend public variables (`VITE_*`, `NEXT_PUBLIC_*`) from backend secrets (`SOLARCH_ADMIN_SECRET`, `NEON_DATABASE_URL`).
- **Zero Exposure**: Never log, print, or store raw secret values in terminal output, markdown artifacts, or committed configuration files.
- **Missing Variable Reporting**: If required environment variables are missing, explicitly report the missing keys to the user instead of injecting placeholder/dummy values.

---

## 7. Execution Checklist & Output Reporting

Upon completion of the deployment workflow, output a structured report:

```markdown
### Solarch Deployment Summary

- **Project Archetype**: [Static SPA | SSR | Worker | API Server | Container]
- **Target Provider**: [Cloudflare | Vercel | Render | Fly.io]
- **Environment**: [Preview | Production]
- **Deployment URL**: [URL]
- **Git Commit**: [Commit Hash]

#### Post-Deployment Health Verification
(Delegated to `solarch-deployment-verify`)
```
