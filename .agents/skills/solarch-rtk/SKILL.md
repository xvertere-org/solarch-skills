---
name: solarch-rtk
description: Execute token-efficient development workflows, compress noisy shell and tool outputs, and minimize agent context consumption across Solarch repositories using RTK (Rust Token Killer) patterns. Use when running repetitive Git operations, inspecting large test logs, filtering TypeScript compilation output, condensing search results, or setting up token-efficient developer workflows. Trigger on queries like "Use RTK to reduce the noisy output from our test commands", "How can we reduce the amount of shell output the Solarch agent reads?", "Set up a token-efficient workflow without hiding important failures", "Our test suite prints 20,000 lines of output", or "Make this repository easier for an AI agent to navigate because every command dumps huge amounts of output". Do NOT trigger on requests to write Rust code, create Rust CLIs, or build Rust components for Solarch.
---

# Solarch RTK (Token-Efficient Agent Workflow)

## Purpose

Guide agents and developers in using RTK (Rust Token Killer) patterns to compress noisy shell and command outputs before they enter LLM context, preventing context exhaustion while strictly preserving critical error diagnostics, stack traces, and failure signals.

---

## References

### Ecosystem Foundations
- [Solarch Architecture & Component Boundaries](../references/solarch-architecture.md)
- [Solarch Engineering Principles](../references/engineering-principles.md)

### RTK Methodologies
- [RTK Command Patterns for Solarch Development](references/command-patterns.md)
- [Token-Efficient Workflow & Cost Math Guide](references/token-efficiency-guide.md)

---

## Tool Independence: RTK vs. Solarch Ecosystem

RTK is an optional, external developer productivity CLI proxy.

- **Not a Runtime Dependency**: RTK is **never** added to `package.json`, `@solarch/core-client`, or Solarch Server dependencies.
- **No Behavioral Modification**: RTK filters standard output formatting; it does not alter application runtime logic, API contracts, or exit codes.
- **Graceful Fallback**: If RTK is not installed on the system, **never fail the task**. Execute standard native commands (`git status`, `npm test`, `npx tsc`) directly.

---

## Core Execution Discipline: Progressive Investigation

When navigating Solarch codebases, investigate progressively from macro summary to high-fidelity detail:

```text
1. Compact Overview (rtk git status / rtk ls)
        ↓
2. Isolate Failures (rtk test <cmd> / rtk tsc / rtk lint)
        ↓
3. Narrow File & Line Scope (rtk grep / rtk diff)
        ↓
4. Escalate to Full Context (rtk proxy <cmd> / raw command)
```

1. **Start with Compact Output**: Run commands that group and compress outputs (`rtk git status`, `rtk tsc`, `rtk vitest`) to avoid filling context with thousands of passing lines.
2. **Focus on Failures**: Rely on test filters that collapse successful cases into counts and highlight failing assertions.
3. **Escalate Immediately When Needed**: If an error message is truncated, a stack trace is partial, or a contract violation is unclear, immediately re-run with `rtk proxy <cmd>` or raw shell commands.

---

## Safety Rules: Correctness Over Token Savings

1. **Full Diagnostic Fidelity**: When debugging production issues, complex race conditions, or unhandled exceptions, do not suppress stack traces. Full diagnostic context takes precedence over token minimization.
2. **Pre-Commit Diffs**: Before executing high-impact commits (database migrations, security rule modifications), inspect complete diffs rather than relying solely on condensed headers.
3. **Contradictory Exit Codes**: If a command exits with a non-zero code but output appears empty due to aggressive filtering, re-run immediately with `rtk proxy <cmd>`.
4. **Accurate Cost Attribution**: A 60%–90% reduction in bash output does not translate to a 60%–90% reduction in total LLM costs. Do not make misleading claims regarding billing impact.

---

## Cross-Skill Boundaries

- **`solarch-rtk`**: Owns command output efficiency, shell output filtering, and progressive exploration workflows.
- **`solarch-testing`**: Owns test strategy, test layer design (unit, contract, integration), and assertion definitions. (When tests are noisy, `solarch-rtk` compresses output while `solarch-testing` designs the tests).
- **`solarch-code-quality`**: Owns architectural and code quality audits.
- **Rust Development Exclusion**: Requests to write Rust code, implement Rust SDK components, or build Rust CLIs must **never** route to `solarch-rtk`.

---

## Diagnostic Output Format

When applying RTK workflows, present results emphasizing actionable signals:
- **Command Executed**: The filtered command run (e.g. `rtk vitest`).
- **Filtered Summary**: Total tests run, passing count collapsed, failed assertion details.
- **Actionable Failures**: File paths, line numbers, and error messages without boilerplate logs.
- **Bypass Hint**: Mention `rtk proxy <cmd>` if full uncompressed output is needed for subsequent inspection.
