---
name: solarch-testing
description: Strategy and implementation guide for testing Solarch components across unit, contract, integration, security, and E2E layers. Use when writing tests, identifying test coverage gaps, designing test strategies, or verifying observable behavior. Trigger on queries like "add tests", "what tests do we need?", "test this feature", "improve coverage", "write integration tests", or "verify this behavior".
---

# Solarch Testing Strategy & Implementation

## Purpose

Define testing strategies and implement high-confidence, minimal test suites for Solarch components focused on observable behavior rather than internal implementation details.

---

## Shared References

- [Solarch Architecture & Boundaries](../references/solarch-architecture.md)
- [Solarch Engineering Principles](../references/engineering-principles.md)

---

## Core Testing Philosophy

- **Test Observable Behavior**: Assert on public API inputs/outputs, network protocols, events, and database records — not private variables or mock function call counts.
- **Contract Conformance Over Mocks**: For Solarch protocol testing, prefer real server/contract conformance tests over mock server implementations whenever practical.
- **Smallest Effective Suite**: Avoid creating redundant test cases. Focus on high-value boundary conditions, happy paths, and critical failure modes.

---

## Testing Layers

1. **Unit**: Pure functions, query builders, serialization/deserialization logic, field validation rules.
2. **Contract**: Protocol compatibility between `@solarch/core-client` and Solarch Server REST/Realtime contracts.
3. **Integration**: Database driver operations (SQLite/Postgres/Neon), authentication flows, collection migration execution, file storage handlers.
4. **Security**: Collection rule enforcement, unauthorized access rejections, secret sanitization in responses.
5. **E2E**: Complete flow from platform SDKs (e.g., React hook) through Core Client to Solarch Server.
6. **Package Verification**: Package export verification (ESM/CJS bundles, type definitions, node vs browser entry points).

---

## Feature-Specific Verification Focus

- **Authentication**: Valid login, expired token refresh, invalid credential rejection, logout state cleanup.
- **Collections & Records**: Filtering, pagination bounds, schema validation failures, collection permission checks (negative path).
- **Realtime**: Connection establishment, subscription filtering, disconnect/reconnect handling, authorization failures.
- **Migrations & CLI**: Idempotent schema migrations, rollbacks, CLI flag parsing, non-zero exit codes on failure.

---

## Implementation Guidelines

- Always include negative-path test cases (e.g., unauthorized request returns 403, invalid input returns 400 with canonical error JSON).
- Ensure test teardown releases database locks, closes WebSocket connections, and cleans up temporary files.
