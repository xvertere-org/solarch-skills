# RTK Command Patterns for Solarch Development

When RTK is installed in the local developer environment, use these command patterns across Solarch repositories to compress verbose shell outputs. If RTK is not installed or when full unfiltered output is required, execute standard shell commands directly.

---

## 1. Git Operations

Verbose Git commands generate excessive log lines that consume agent context windows without providing actionable signals.

| Task | RTK Pattern | Standard Fallback | Filter Behavior |
| :--- | :--- | :--- | :--- |
| **Status** | `rtk git status` | `git status` | Groups changed/staged files compactly by state. |
| **Diff** | `rtk git diff` | `git diff` | Truncates unchanged context lines and strips file headers. |
| **Commit Log** | `rtk git log -n 10` | `git log -n 10 --oneline` | Compact commit hash, author, and subject. |
| **Stage & Commit** | `rtk git add . && rtk git commit -m "msg"` | `git add . && git commit -m "msg"` | Collapses progress output to a one-line confirmation. |

---

## 2. TypeScript & Build Operations

Solarch projects (`Solarch Server`, `@solarch/core-client`, Platform SDKs) rely on TypeScript compilation and linters.

| Task | RTK Pattern | Standard Fallback | Filter Behavior |
| :--- | :--- | :--- | :--- |
| **Typecheck** | `rtk tsc` | `npx tsc --noEmit` | Groups compilation errors by file; silent on pass. |
| **Linting** | `rtk lint` or `rtk lint biome` | `npm run lint` | Groups violations by rule and file location. |
| **Next.js Build** | `rtk next build` | `npm run build` | Strips static route generation noise, shows route table/errors. |
| **Cargo Build** | `rtk cargo build` | `cargo build` | Filters compilation status lines; shows errors/warnings only. |
| **Clippy Lints** | `rtk cargo clippy` | `cargo clippy` | Collapses boilerplate compiler lines, isolates actionable lints. |

---

## 3. Test Suites & Diagnostics

Large test suites often emit tens of thousands of lines of output for passing tests. RTK provides failure-focused filtering:

| Task | RTK Pattern | Standard Fallback | Filter Behavior |
| :--- | :--- | :--- | :--- |
| **Vitest** | `rtk vitest` | `npx vitest run` | Collapses passing tests into counts; shows failure diffs only. |
| **Jest** | `rtk jest` | `npm test` | Suppresses passing spec outputs; isolates failed assertions. |
| **Playwright E2E** | `rtk playwright test` | `npx playwright test` | Emits failure traces only; silences passing browser steps. |
| **Generic Test** | `rtk test <cmd>` | `<cmd>` | Universal failure extractor: strips progress, outputs failures. |
| **Error Extractor** | `rtk err <cmd>` | `<cmd>` | Captures stderr and error logs from arbitrary scripts. |

---

## 4. Repository & File Exploration

When exploring unfamiliar subdirectories or large source trees:

| Task | RTK Pattern | Standard Fallback | Filter Behavior |
| :--- | :--- | :--- | :--- |
| **Directory Tree** | `rtk ls <dir>` | `ls -la <dir>` | Hierarchical tree with file counts instead of raw lists. |
| **Code Structure** | `rtk read <file>` | `cat <file>` | Extracts function signatures, classes, and types. |
| **Heuristic Summary** | `rtk smart <file>` | `head -n 50 <file>` | 2-line heuristic structure summary. |
| **Grep / Search** | `rtk grep <pattern> <dir>` | `rg <pattern> <dir>` | Groups matches by file, truncating long matching lines. |
| **Find Files** | `rtk find "<glob>" <dir>` | `find <dir> -name "<glob>"` | Compact listing of matched file paths. |

---

## 5. Unfiltered Execution & Bypass Rules

**Never compromise diagnostic accuracy for token savings.** When full context is required:

- **Raw Bypass with Tracking**:
  ```bash
  rtk proxy <command> [args...]
  ```
  Runs the underlying command with **100% unfiltered output** while logging execution in the local RTK metrics database.
- **Environment Bypass**:
  ```bash
  RTK_DISABLED=1 <command>
  ```
- **Direct Shell Command**:
  Run standard commands directly (e.g. `git diff -U10`, `npx vitest --run --reporter=verbose`) when debugging stack traces, investigating deep compiler flags, or inspecting intricate diffs.
