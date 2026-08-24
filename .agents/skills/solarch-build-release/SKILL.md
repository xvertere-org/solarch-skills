---
name: solarch-build-release
description: Validate production builds, package assets, declaration files, and npm artifact shapes for Solarch packages and SDKs. Use when diagnosing broken npm builds, checking package exports, or preparing package releases. Trigger on queries like "our npm package build is broken", "verify this package before publishing", "prepare this SDK for release", "check npm pack output", or "make sure the package exports are correct".
---

# Solarch Build & Package Release Verification

## Purpose

Verify production build outputs, TypeScript declaration files, package entry point exports, and npm tarball contents for Solarch core packages, server modules, and platform SDKs prior to publication.

---

## Shared References

- [Solarch Architecture & Boundaries](../references/solarch-architecture.md)
- [Solarch CI/CD Principles](../references/ci-cd-principles.md)

---

## Build Verification Rules

### 1. `npm pack --dry-run` Audit
Always inspect the actual published tarball contents before releasing:
- Confirm `.d.ts` declaration files are generated and included.
- Verify source `.ts` files and test assets are excluded from public npm packages.
- Ensure `README.md`, `LICENSE`, and `package.json` are cleanly packaged.

### 2. Package Exports & Dual Module Safety
- Check `package.json` fields (`main`, `module`, `types`, `exports`).
- Verify ESM and CJS entry points resolve correctly without dual-package hazards.
- Confirm `@solarch/core-client` does not bundle platform-specific native binaries or Node `fs` references into web/client exports.

### 3. Tree-Shaking & Bundle Size
- Verify SDK bundles support tree-shaking and omit unnecessary development utilities or heavy third-party helper libraries.

### 4. Solarch Server Assets
- Ensure DB migration scripts, configuration templates, and CLI binaries are correctly included in server release builds.

---

## Non-Publication Rule

> **Never assume a successful TypeScript compilation (`tsc`) equals a publishable package.**
Always execute dry-run packaging and verify exported type definitions before publishing.
