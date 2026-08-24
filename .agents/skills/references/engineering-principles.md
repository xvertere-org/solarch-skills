# Solarch Engineering & Ponytail Philosophy

## The Ponytail / Lazy-Senior-Developer Mindset

In the Solarch ecosystem, we value high correctness, minimal code, strong contracts, and long-term maintainability. We follow the **Ponytail (Lazy Senior Developer)** philosophy:

> **Understand the problem deeply first, then choose the smallest correct solution.**

### 1. Hierarchy of Implementation Options

Before writing new code, evaluate solutions in this exact order:

```text
Existing codebase implementation
        ↓
Existing Solarch abstraction / utility
        ↓
Standard language library / built-ins
        ↓
Native platform API
        ↓
Existing direct dependency
        ↓
Minimum necessary new code
```

### 2. Core Discipline Principles

- **Inspect Before Modifying**: Trace the exact execution path and data flow before editing files.
- **Prefer Deletion Over Addition**: Removing dead or redundant code is better than writing more logic.
- **Avoid Speculative Abstractions**: Do not build for hypothetical future requirements ("YAGNI").
- **Question Unnecessary Complexity**: Challenge requests or designs that introduce complex wrappers, layers, or generic abstractions when a simple function suffices.
- **Preserve Frozen Contracts**: Never casually break public APIs, database abstractions, or protocol schemas.

---

## Crucial Distinction: Minimal Implementation vs. Unsafe Shortcut

A **minimal implementation** reduces unnecessary code while maintaining full system integrity. An **unsafe shortcut** cuts corners on critical safety mechanisms.

### MUST NEVER be removed or bypassed to reduce code size:

1. **Security & Authorization**: Server-side collection rules, token validation, permission checks.
2. **Validation**: Input sanitization, schema verification, boundary type checks.
3. **Data-Loss Protection**: Transactions, migration integrity, storage flushing.
4. **Error Handling**: Graceful error handling, resource cleanup, canonical error propagation.
5. **Lifecycle Correctness**: Subscription cleanup, async cancellation, race condition prevention.
6. **Type Safety & Testing**: Sound TypeScript types and meaningful test verification.
