# Solarch CI/CD Principles & Pipeline Architecture

## 1. Pipeline Philosophy

In the Solarch ecosystem, Continuous Integration and Continuous Delivery (CI/CD) serve to guarantee correctness, security, contract stability, and production readiness.

> **A green pipeline must be meaningful.**

### Core Rules

1. **Proportional Validation**: Determine the smallest sufficient validation pipeline based on the scope of changes (e.g., documentation vs. `@solarch/core-client` protocol change vs. full release).
2. **Strict Failure Classification**: Never automatically classify unexplained failures as "flaky" or "environmental" without concrete evidence. Distinguish transient infrastructure failures from deterministic code/contract failures.
3. **No Safety Bypasses**: Never recommend skipping failing tests, weakening assertions, disabling security checks, or bypassing contract tests to force a green pipeline.
4. **Least Privilege & Secret Protection**: Workflows and pipelines must strictly enforce minimum required permissions and protect credentials from untrusted PR execution or log exposure.

---

## 2. Standard Solarch Pipeline Stages

```text
Change Detection & Scope Analysis
        ↓
Dependency Installation (Deterministic / Locked)
        ↓
Static Analysis & Formatting Check
        ↓
Typecheck (Strict TypeScript)
        ↓
Unit & Local Module Tests
        ↓
Integration & Canonical Contract Tests
        ↓
Security & Dependency Vulnerability Audit
        ↓
Production Build & Asset Packaging Verification (`npm pack --dry-run`)
        ↓
End-to-End (E2E) & Distribution Verification (Where Applicable)
        ↓
Release & Version Bumping (Semantic Versioning)
        ↓
Deployment & Post-Deploy Smoke Verification
```

---

## 3. Failure Classification Guide

When a pipeline fails, classify the root cause into one of these distinct categories before taking action:

- **Code Failure**: Compilation error, syntax flaw, or logic bug in active source code.
- **Test Failure**: Assertion failure in unit or integration test suite representing a broken contract or regression.
- **Contract Failure**: Incompatibility between `@solarch/core-client`, Solarch Server REST/WS APIs, or DB schemas.
- **Security Failure**: Secret leak detected, high-severity CVE in dependency, or broken authorization check.
- **Dependency Failure**: Missing dependency, package lock mismatch, or ESM/CJS resolution failure.
- **Infrastructure / Provider Failure**: Network timeout, runner out-of-memory, or registry outage.
- **CI Configuration Failure**: Malformed YAML, invalid step syntax, or missing environment variables.
- **Flaky Test**: Non-deterministic test failure verified by intermittent pass/fail history on unchanged code.
