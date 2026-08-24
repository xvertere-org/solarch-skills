---
name: solarch-dependency-audit
description: Audit direct and dev dependencies in Solarch packages for necessity, security, compatibility, bloat, and platform constraints. Use when evaluating npm packages, auditing bundle size, checking vulnerability reports, or pruning dependencies. Trigger on queries like "audit dependencies", "do we need this package?", "remove unnecessary dependencies", "check dependency security", or "why are there so many packages?".
---

# Solarch Dependency Audit

## Purpose

Audit direct and development dependencies in Solarch repositories to ensure minimal package footprint, prevent platform leakage, reduce security vulnerabilities, and eliminate unnecessary third-party risk.

---

## Shared References

- [Solarch Architecture & Boundaries](../references/solarch-architecture.md)
- [Solarch Engineering Principles](../references/engineering-principles.md)

---

## Core Audit Questions

For every existing or proposed dependency, answer:

1. **Actual Usage**: Is the dependency imported in active code, or is it unused dead weight?
2. **Existing Abstractions**: Can existing Solarch code or helper functions replace this dependency?
3. **Standard Library**: Can standard language features (e.g., native `fetch`, `structuredClone`, `URLPattern`, `Crypto`) solve the problem?
4. **Existing Dependencies**: Is another installed package already providing this capability?
5. **Platform Constraints**: Does this dependency introduce Node.js or browser-specific assumptions into `@solarch/core-client`?
6. **Cost vs. Value**: Is the security, maintenance, and bundle size cost justified by the functionality provided?

---

## Solarch Package Audit Guidelines

- **`@solarch/core-client`**: Must maintain minimal dependencies. Zero dependencies with Node.js `fs`/`path` or DOM assumptions.
- **Platform SDKs**: Verify dependencies do not duplicate `@solarch/core-client` logic or bundle unnecessary heavy transitive utilities.
- **Solarch Server**: Ensure database driver dependencies (SQLite native bindings, Postgres client libs) are scoped to server/driver packages and never leak into client exports.
- **Bundle & Compatibility**: Check ESM/CJS compatibility, dual-package hazard risks, license compliance, and tree-shaking support.

---

## Guidance on Removal

- **Do NOT** recommend removing a dependency solely because it is small or unfamiliar without auditing actual usage and requirements.
- When replacing a package, provide a concrete code snippet demonstrating how native code or existing Solarch utilities replace it cleanly.
