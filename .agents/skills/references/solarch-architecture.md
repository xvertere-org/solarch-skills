# Solarch Architecture & Component Boundaries

## 1. Solarch Ecosystem Architecture

The canonical Solarch data and execution flow is structured in strict layers:

```text
Solarch Server (REST / Realtime / Auth / DB Abstraction)
        ↓
canonical protocol (JSON over HTTP/WS)
        ↓
@solarch/core-client (Platform-independent logic & client protocol)
        ↓
platform SDKs (@solarch/react, @solarch/react-native, @solarch/tauri, etc.)
        ↓
User Applications
```

### Key Architectural Boundaries

1. **Solarch Server**:
   - Manages relational database drivers (SQLite, PostgreSQL, Neon PostgreSQL) via the `DatabaseDriver` abstraction layer.
   - Enforces authentication, authorization collection rules, schema validation, migrations, file storage, and realtime event dispatching.
   - Provider-specific database representations must **never** leak directly into public REST/WebSocket contracts. Data must be serialized to canonical JSON before returning to clients.

2. **`@solarch/core-client`**:
   - Must remain **strictly platform-independent**.
   - Must **never** reference browser globals (`window`, `localStorage`, DOM), Node.js built-ins (`fs`, `path`), platform APIs, React/UI frameworks, or database drivers.
   - Contains core state management, query building, canonical API request handling, authentication state machines, and realtime subscription multiplexing.

3. **Platform SDKs**:
   - Build on top of `@solarch/core-client`.
   - Provide platform-specific bindings (e.g., React hooks, React Native persistent storage adapters, Tauri IPC bridges).
   - Must **never** reimplement core client serialization, query building, or protocol handling.

4. **Solarch Admin**:
   - First-party administration UI application.
   - Must consume `@solarch/core-client` or official SDK abstractions wherever possible.
   - Must not invent parallel or ad-hoc API models for core server endpoints.

---

## 2. Frozen Contracts & Foundations

Unless explicitly modified by an authorized architectural RFC, the following contracts are frozen:

- **Database Abstraction**: `DatabaseDriver`, capability model, `ResolvedAppConfig`, external DB configurations.
- **Canonical API Protocol**: REST endpoints, standard error responses, query parameter serialization, realtime WebSocket event schemas.
- **Security & Authorization**: Collection access rules evaluated server-side. Client-side checks are UI hints, never security boundaries.

### Database Driver Implementation Statuses

- **SQLite, PostgreSQL, Neon PostgreSQL**:
  - **Status**: Implemented / Active
  - **Evidence**: `DatabaseDriver` capability model and frozen core database contracts in Solarch Server.
- **MongoDB**:
  - **Status**: Unknown / requires verification
  - **Evidence**: Available workspace skill references do not contain core driver source implementations or verified DatabaseDriver contracts for MongoDB.

