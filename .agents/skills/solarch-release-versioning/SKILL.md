---
name: solarch-release-versioning
description: Enforce semantic versioning and public contract compatibility across Solarch packages, APIs, SDKs, and server releases. Use when evaluating release version bumps, checking public breaking changes, or preparing release metadata. Trigger on queries like "what version should this release be?", "is this a breaking change?", "prepare the next Solarch release", or "check whether this needs a major version".
---

# Solarch Release Versioning & Contract Compatibility

## Purpose

Enforce semantic versioning (SemVer) and public contract compatibility across Solarch packages, platform SDKs, `@solarch/core-client`, REST/WS protocols, and database schema migrations.

---

## Shared References

- [Solarch Architecture & Boundaries](../references/solarch-architecture.md)
- [Solarch CI/CD Principles](../references/ci-cd-principles.md)

---

## SemVer Decision Framework

Determine version increments strictly based on public API and protocol contracts:

```text
Patch (0.0.X)  → Backward-compatible bug fixes, performance improvements, internal refactors.
Minor (0.X.0)  → Backward-compatible new features, new REST endpoints, non-breaking SDK additions.
Major (X.0.0)  → Breaking changes to public APIs, Core Client contracts, DB migration structures, or removed exports.
```

---

## Solarch Contract Audit Checklist

Before recommending a version bump, check for breaking changes across all public layers:

### 1. `@solarch/core-client` & SDKs
- Have any exported interfaces, types, methods, or parameters been removed, renamed, or made mandatory?
- Have default method signatures or return types changed?
- Have peer dependency requirements been bumped restrictively?

### 2. Solarch Server & Protocol
- Have any canonical REST endpoint paths, JSON response formats, query parameters, or WS event payloads changed incompatibly?
- Do database migration changes require manual intervention or data transformation?

### 3. Solarch CLI
- Have CLI commands, flags, or configuration file key names been altered or removed?

---

## Key Rule

> **Internal refactors do NOT warrant major version bumps. Public type removals, protocol breaks, or mandatory config additions DO.**
