---
name: solarch-ponytail
description: Apply Solarch-specific Ponytail engineering discipline to produce minimal, correct, and high-quality solutions. Use when simplifying code, reducing complexity, eliminating overengineering, reviewing implementation approaches, or writing minimal new code in Solarch codebases. Trigger on queries like "simplify this", "clean this up", "make this implementation smaller", "is this overengineered?", "review this approach", or "implement this with minimal code".
---

# Solarch Ponytail Engineering Discipline

## Purpose

Guide agents to act like careful senior Solarch engineers who eliminate unnecessary code, avoid overengineering, and select the smallest correct solution while maintaining strict safety, correctness, and architectural integrity.

---

## Shared References

- [Solarch Architecture & Boundaries](../references/solarch-architecture.md)
- [Solarch Engineering Principles](../references/engineering-principles.md)

---

## Core Execution Rules

### 1. Inspect Before Modifying
- Trace the actual data flow and execution path across Solarch boundaries before proposing or writing code.
- Locate existing utilities or abstractions in `@solarch/core-client` or Solarch Server instead of inventing new helpers.

### 2. Implementation Hierarchy
Always choose the first available tier:
1. Use existing codebase functions/abstractions.
2. Use standard library or language built-ins.
3. Use native platform APIs.
4. Use an already installed dependency.
5. Write the smallest possible new implementation.

### 3. Simplify & Prune
- **Prefer deletion**: Eliminating redundant wrapper functions or dead code paths is preferable to adding configuration options.
- **Reject speculative generality**: Write code strictly for current, verified requirements.
- **Avoid duplicate protocols**: Ensure realtime, querying, or serialization logic isn't reinvented locally when canonical mechanisms exist.

---

## Critical Boundary: Minimal vs. Unsafe

When simplifying code, **NEVER** remove or weaken:
- Server-side authorization or rule checks
- Input validation and schema sanitization
- Data-loss protection (transactions, storage flushing)
- Error handling, timeouts, and resource cleanup
- Type safety and meaningful test coverage

---

## Verification & Stop Criteria

- **Verification**: Run existing tests or execute build checks to ensure no regressions were introduced.
- **Stop Criteria**: Stop as soon as the smallest correct solution is implemented and verified. Do not continue refactoring adjacent untouched areas unless explicitly requested.
