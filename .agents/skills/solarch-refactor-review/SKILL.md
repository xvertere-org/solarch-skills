---
name: solarch-refactor-review
description: Evaluate proposed or completed refactoring in Solarch codebases to assess value, architectural risk, and necessity. Use when evaluating structural changes, abstractions, or codebase reorganization. Trigger on queries like "should we refactor this?", "review this refactor", "is this abstraction worth it?", or "can we clean up this architecture?".
---

# Solarch Refactor Review

## Purpose

Assess proposed or implemented refactoring in Solarch repositories to answer the fundamental question: **"Is this refactor actually worth doing?"**

---

## Shared References

- [Solarch Architecture & Boundaries](../references/solarch-architecture.md)
- [Solarch Engineering Principles](../references/engineering-principles.md)

---

## Evaluation Criteria

Before approving or executing a refactor, evaluate:

1. **Concrete Value**: Does this fix a bug, significantly reduce complexity, improve performance, or eliminate proven duplication?
2. **Contract Stability**: Does the refactor preserve frozen database abstractions, `@solarch/core-client` interfaces, and server REST/WS protocols?
3. **Scope & Blast Radius**: What components in the dependency graph are affected? Are platform SDKs or Admin impacted?
4. **Maintenance Cost**: Does the change introduce new abstractions, generics, or configurations that add cognitive overhead for future maintainers?
5. **Testing Burden**: Can existing tests verify the refactor, or does it require rewriting the test suite?

---

## Automatic Rejection Criteria

Reject refactoring proposals that:
- Are purely cosmetic or stylistic without functional or clarity benefits.
- Introduce speculative abstractions or unnecessary wrapper layers.
- Break public API or core protocol contracts without a formal RFC.
- Combine refactoring with unrelated feature additions or bug fixes (scope creep).
- Replace clear imperative logic with complex generic abstractions.

---

## Recommended Execution Strategy

When a refactor is justified, mandate a small, incremental approach:

```text
Small incremental refactor step
        ↓
Run automated test suite
        ↓
Verify runtime & contract compatibility
        ↓
Proceed to next step
```

*Never accept giant rewrites in a single monolithic change.*
