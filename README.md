# Solarch Skills v1

The canonical repository of **Google Antigravity (AGY)** skills for the Solarch ecosystem.

This repository provides reusable, minimal, and high-correctness agent workflows, architectural guardrails, automated CI/CD patterns, deployment orchestrators, and performance tooling designed specifically for **Solarch Server**, **`@solarch/core-client`**, **Platform SDKs**, and end-user applications.

---

## 🏛️ Core Philosophy & Principles

Solarch skills adhere to the **Ponytail (Lazy Senior Developer)** engineering discipline:

> **"Understand the problem deeply first, then choose the smallest correct solution."**

### Guiding Tenets
- **Every Skill Must Earn Its Existence**: We avoid redundant, speculative, or overly generic skills. Workflows must represent proven engineering practices.
- **Progressive Disclosure**: SKILL definitions are kept concise and actionable. Heavy technical details, provider-specific manuals, and architectural diagrams are isolated in `references/`.
- **Contract Stability**: Public REST/WebSocket protocols, `@solarch/core-client` isolation rules, and `DatabaseDriver` abstractions are frozen and protected.
- **Verification Over Assumptions**: CLI exit code `0` never implies health. Performance, deployments, and builds must be proven with empirical verification.

---

## 📦 Skills Directory (`.agents/skills/`)

The repository currently maintains **19 canonical skills** categorized into four core engineering domains:

### 1. Core Engineering & Architecture

| Skill | Description | Primary Triggers |
| :--- | :--- | :--- |
| [`solarch-ponytail`](.agents/skills/solarch-ponytail/SKILL.md) | Produces minimal, correct code while eliminating unnecessary abstractions and wrappers. | *"simplify this"*, *"is this overengineered?"*, *"clean this up"* |
| [`solarch-code-quality`](.agents/skills/solarch-code-quality/SKILL.md) | Audits code for architectural compliance, TypeScript strictness, async leaks, and boundary isolation. | *"review this PR"*, *"is this production ready?"*, *"audit code"* |
| [`solarch-refactor-review`](.agents/skills/solarch-refactor-review/SKILL.md) | Evaluates whether structural refactorings are justified, assessing complexity vs. tangible benefit. | *"should we refactor this?"*, *"is this abstraction worth it?"* |
| [`solarch-security-review`](.agents/skills/solarch-security-review/SKILL.md) | Audits server-side authorization collection rules, token handling, multi-tenant isolation, and RBAC. | *"security audit"*, *"check collection rules"*, *"is this secure?"* |
| [`solarch-dependency-audit`](.agents/skills/solarch-dependency-audit/SKILL.md) | Audits npm dependencies for bloat, necessity, licenses, and ensures zero platform leakage into Core Client. | *"audit dependencies"*, *"do we need this package?"*, *"prune deps"* |

### 2. Testing, Release & CI/CD

| Skill | Description | Primary Triggers |
| :--- | :--- | :--- |
| [`solarch-testing`](.agents/skills/solarch-testing/SKILL.md) | Designs high-confidence test suites across unit, contract, integration, and E2E layers. | *"add tests"*, *"test this feature"*, *"contract tests"*, *"coverage"* |
| [`solarch-ci-cd`](.agents/skills/solarch-ci-cd/SKILL.md) | Orchestrates repository pipelines, test matrices, and classifies failures (code vs. infra vs. flaky). | *"fix our CI"*, *"design CI pipeline"*, *"why is pipeline failing?"* |
| [`solarch-github-actions`](.agents/skills/solarch-github-actions/SKILL.md) | Generates and debugs hardened GitHub Actions workflows with least-privilege permissions. | *"create workflow"*, *"fix GitHub Actions"*, *"add matrix job"* |
| [`solarch-build-release`](.agents/skills/solarch-build-release/SKILL.md) | Validates build artifacts, declaration maps, ESM/CJS exports, and `npm pack` tarball shapes. | *"verify package exports"*, *"broken npm build"*, *"prepare SDK release"* |
| [`solarch-release-versioning`](.agents/skills/solarch-release-versioning/SKILL.md) | Enforces Semantic Versioning (SemVer) and detects breaking changes in public API contracts. | *"what version bump?"*, *"is this a breaking change?"*, *"release version"* |

### 3. Deployment & Cloud Hosting

| Skill | Description | Primary Triggers |
| :--- | :--- | :--- |
| [`solarch-deployment`](.agents/skills/solarch-deployment/SKILL.md) | Master orchestrator; detects application archetypes and coordinates the 13-stage deployment lifecycle. | *"deploy this project"*, *"how should we deploy?"*, *"setup deployment"* |
| [`solarch-deployment-verify`](.agents/skills/solarch-deployment-verify/SKILL.md) | Post-deployment runtime smoke verification; ensures HTTP reachability and health independent of CLI exit codes. | *"verify deployment"*, *"is app working?"*, *"CLI returned 0 but 500"* |
| [`solarch-cloudflare-deploy`](.agents/skills/solarch-cloudflare-deploy/SKILL.md) | Deploys Solarch Workers, Pages, and Edge APIs via Cloudflare Wrangler. | *"deploy to Cloudflare"*, *"use Wrangler"*, *"deploy to Workers"* |
| [`solarch-vercel-deploy`](.agents/skills/solarch-vercel-deploy/SKILL.md) | Deploys Next.js, static frontends, and SSR web applications to Vercel infrastructure. | *"deploy to Vercel"*, *"host frontend on Vercel"* |
| [`solarch-render-deploy`](.agents/skills/solarch-render-deploy/SKILL.md) | Deploys persistent Solarch Server web services and PostgreSQL instances on Render. | *"deploy to Render"*, *"create Render service"* |
| [`solarch-fly-deploy`](.agents/skills/solarch-fly-deploy/SKILL.md) | Deploys containerized Solarch API servers and microservices to Fly.io via `flyctl`. | *"deploy to Fly.io"*, *"setup fly.toml"* |

### 4. Performance, Productivity & Metaprogramming

| Skill | Description | Primary Triggers |
| :--- | :--- | :--- |
| [`solarch-api-scaler`](.agents/skills/solarch-api-scaler/SKILL.md) | Measures API throughput, diagnoses saturation bottlenecks, optimizes multi-core compute, and evaluates scaling costs. | *"how many RPS can this handle?"*, *"API bottleneck"*, *"should we add Redis?"* |
| [`solarch-rtk`](.agents/skills/solarch-rtk/SKILL.md) | Executes token-efficient agent workflows using RTK (Rust Token Killer) patterns to compress verbose shell outputs. | *"reduce shell output"*, *"noisy test logs"*, *"token-efficient workflow"* |
| [`solarch-skill-creator`](.agents/skills/solarch-skill-creator/SKILL.md) | Designs, evaluates, tests, and packages new Antigravity skills following the 7-question decision gate. | *"create a skill"*, *"turn this into a skill"*, *"add skill evals"* |

---

## 📚 Shared Architectural References (`.agents/skills/references/`)

All skills link to canonical foundation documents rather than duplicating context:

- [`solarch-architecture.md`](.agents/skills/references/solarch-architecture.md): Strict layer boundaries between Solarch Server, `@solarch/core-client`, Platform SDKs, and verified database drivers (SQLite, PostgreSQL, Neon PostgreSQL).
- [`engineering-principles.md`](.agents/skills/references/engineering-principles.md): The Ponytail philosophy, implementation priority hierarchy, and the critical distinction between minimal code and unsafe shortcuts.
- [`ci-cd-principles.md`](.agents/skills/references/ci-cd-principles.md): Pipeline stages, proportional validation, and failure classification taxonomies.
- [`deployment-principles.md`](.agents/skills/references/deployment-principles.md): Archetype detection matrix, environment isolation, and the 13-stage deployment lifecycle.

---

## 🧪 Evaluation Suite (`evals/evals.json`)

To ensure skills trigger accurately and respect cross-skill boundaries without collisions, the repository maintains an automated benchmark suite of **82 evaluations**:

- **Single-Skill Evals**:
  - *Obvious*: Explicit domain requests testing baseline trigger accuracy (`expected_trigger: true`).
  - *Complex*: Multi-step, realistic engineering challenges (`expected_trigger: true`).
  - *Near-Miss*: Adjacent or ambiguous queries that must **not** trigger the skill (`expected_trigger: false`).
- **Cross-Skill Collision Evals**:
  - Prompts where multiple skills overlap, asserting `expected_primary`, `expected_supporting`, and `must_not_trigger` boundaries.

---

