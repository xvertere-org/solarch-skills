---
name: solarch-ci-cd
description: Orchestrate, design, inspect, and debug CI/CD pipelines and automated test execution matrices for Solarch repositories. Use when configuring repository pipelines, analyzing pipeline failures, structuring CI test matrices (unit, contract, integration, E2E), or determining PR vs release validation gates. Trigger on queries like "design our CI pipeline", "fix our CI", "why is our pipeline failing?", "set up CI for this Solarch repo", "what tests should CI run?", "add these tests to CI", "our integration tests aren't running in CI", or "what should run on PR vs release?".
---

# Solarch CI/CD Pipeline & Test Matrix Orchestration

## Purpose

Guide agents to design, analyze, debug, and optimize CI/CD pipelines and automated test execution matrices across Solarch repositories, ensuring proportional validation gates, strict failure classification, and zero safety bypasses.

---

## Shared References

- [Solarch Architecture & Boundaries](../references/solarch-architecture.md)
- [Solarch Engineering Principles](../references/engineering-principles.md)
- [Solarch CI/CD Principles](../references/ci-cd-principles.md)

---

## Core Execution Workflow

### 1. Scope Analysis & Pipeline Selection
Do not run a monolithic full pipeline for every change. Tailor execution to affected layers:
- **Documentation Only**: Markdown linting & link checks.
- **SDK / Web Package**: Typecheck, unit tests, bundle packaging check (`npm pack`).
- **Core Client (`@solarch/core-client`)**: Full REST/WS protocol contract tests, cross-platform build verification, strict typecheck.
- **Solarch Server**: DB capability tests (SQLite/Postgres/Neon), REST/WS contract tests, security audit, release build.
- **Full Release**: End-to-end suite, packaging verification, versioning check, artifact publishing.

### 2. CI Test Hierarchy & Matrix Design
Structure test execution gates in CI using a parallel matrix strategy:

```text
Unit Tests (Fast execution, pure logic)
        ↓
Contract Conformance Tests (Solarch Server ↔ Core Client canonical protocol)
        ↓
Integration Tests (DB drivers: SQLite, PostgreSQL, Neon, file storage)
        ↓
Security & Rules Tests (Collection authorization, token validation)
        ↓
E2E & Distribution Tests (SDK packaging, entry points)
```

- **Contract Conformance First**: Prioritize real REST/WS protocol conformance suites when changes touch `@solarch/core-client` or Solarch Server endpoints.
- **Parallel Matrix Optimization**: Run matrix jobs across target Node.js LTS versions and supported DB providers in parallel without duplicating identical unit test assertions.
- **Flaky Test Prevention**: Isolate network-dependent or timing-sensitive integration tests into dedicated steps with explicit timeouts and diagnostic logging.
- **Negative Path Execution**: Verify that negative path security tests (e.g., expecting 401/403 HTTP status codes) execute cleanly in CI without failing build steps.

### 3. Failure Diagnosis & Classification
When a pipeline or test step fails:
1. Inspect raw build/test logs without masking errors.
2. Categorize failure strictly: *Code*, *Test*, *Contract*, *Security*, *Dependency*, *Infra*, *Config*, or *Flaky*.
3. Require empirical proof before labeling any failure as "flaky" or "infrastructure-related".

### 4. Safety Non-Bypass Rule
- **NEVER** recommend commenting out failing tests, disabling ESLint/TypeScript checks, or passing `--force` / `--ignore-scripts` to bypass a broken pipeline.
- If a contract test fails, trace the incompatibility between Server and Core Client rather than weakening test assertions.

---

## Stop & Verification Criteria

- **Verification**: Run local validation commands corresponding to the failing CI step (e.g., `npm run test:contract` or `npm run typecheck`).
- **Stop Criteria**: Pipeline design or fix is verified, least-privilege security boundaries are maintained, and all gates are green.
