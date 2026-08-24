---
name: solarch-code-quality
description: Perform a production-grade code quality and architectural review for Solarch repositories. Use when auditing code changes, reviewing PRs, assessing production readiness, or checking architectural compliance across server, core-client, and SDKs. Trigger on queries like "review this code", "audit this implementation", "find code quality issues", "is this production ready?", or "review this PR".
---

# Solarch Code Quality Review

## Purpose

Audit existing or newly proposed Solarch code to identify architecture, TypeScript, runtime, maintainability, and ecosystem-boundary issues, delivering prioritized findings with concrete, actionable fixes.

---

## Shared References

- [Solarch Architecture & Boundaries](../references/solarch-architecture.md)
- [Solarch Engineering Principles](../references/engineering-principles.md)

---

## Review Checklist

### 1. Architectural & Boundary Compliance
- **Core Client Isolation**: Ensure `@solarch/core-client` contains zero platform-specific references (`window`, `localStorage`, `fs`, React, DOM).
- **SDK & Admin Abstractions**: Verify Platform SDKs and Admin consume canonical `@solarch/core-client` utilities without reinventing query building, serialization, or realtime protocols.
- **Database Layer Isolation**: Ensure database driver/provider details (SQLite, Postgres, Neon) do not leak into public REST/client APIs.

### 2. TypeScript Integrity
- Flag usage of `any`, unsafe `as` assertions, weak types, or duplicated interfaces.
- Check for incorrect nullability assumptions, dead type exports, or non-canonical custom error classes.

### 3. Runtime & Async Safety
- **Resource Cleanup**: Verify subscriptions, event listeners, and timers are properly cleaned up on unmount or cancellation.
- **Concurrency**: Detect race conditions, missing `AbortController` cancellation on stale requests, unhandled promise rejections, and missing retry/timeout handling.

### 4. Maintainability & Code Hygiene
- Check for complex nested conditionals, duplicate logic blocks, misleading comments, or dead code paths.
- Avoid nitpicking whitespace, formatting, or trivial lint style rules handled automatically by tools (Prettier/ESLint).

---

## Output Format

Report findings structured strictly by severity:

### 🔴 Critical (Must Fix)
- Boundary violations (e.g., Node `fs` imported in `@solarch/core-client`).
- Memory/resource leaks, unhandled async crashes, broken core contracts.

### 🟠 High (Should Fix)
- Unsafe `any` casts in public signatures, missing request cancellation, duplicate protocol implementations.

### 🟡 Medium (Improvement)
- Unnecessary abstractions, duplicate utility functions, complex conditionals.

Provide a short, direct code diff or snippet for each recommended fix.
