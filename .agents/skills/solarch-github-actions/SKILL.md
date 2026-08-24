---
name: solarch-github-actions
description: Create, review, debug, and secure GitHub Actions workflows for Solarch repositories. Use when writing workflow YAML files, configuring job permissions, setting up action caching, or auditing pipeline security. Trigger on queries like "create a GitHub Actions workflow", "fix this GitHub Actions workflow", "why is this GitHub workflow failing?", or "add a GitHub Actions matrix".
---

# Solarch GitHub Actions Implementation

## Purpose

Implement, debug, and audit GitHub Actions workflows for Solarch repositories, strictly adhering to security best practices, least-privilege permissions, and clean job orchestration.

---

## Shared References

- [Solarch Architecture & Boundaries](../references/solarch-architecture.md)
- [Solarch CI/CD Principles](../references/ci-cd-principles.md)

---

## Security & Workflow Design Guidelines

### 1. Least Privilege Permissions
Always define explicit top-level `permissions` block in workflow files:
```yaml
permissions:
  contents: read
```
Never use default write-all permissions. Grant elevated permissions (`packages: write`, `id-token: write`) only to specific release/deployment jobs.

### 2. Secret & Fork Protection
- **PR Security**: Never pass production secrets to `pull_request` triggers from public forks. Use `pull_request_target` with extreme caution and never execute untrusted PR code with elevated tokens.
- **Secret Exposure**: Avoid printing environment variables or credentials in workflow step debug scripts.

### 3. Action Pinning & Deterministic Builds
- Pin third-party GitHub Actions to explicit full commit SHA hashes or verified release tags (e.g., `actions/checkout@v4`).
- Use deterministic dependency installation (`npm ci` instead of `npm install`).

### 4. Caching & Efficiency
- Utilize `actions/setup-node` built-in `cache: 'npm'` to accelerate pipeline runs.
- Set explicit job timeouts (`timeout-minutes: 15`) and concurrency groups to automatically cancel obsolete PR runs:
```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```
