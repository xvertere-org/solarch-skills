# Scaling Strategies for Solarch APIs (Cheapest to Most Expensive)

Apply scaling strategies in strict order from lowest cost/effort to highest. Re-benchmark after each isolated intervention.

```text
1. Framework & Runtime Tuning (Free, Code/Config)
        ↓
2. Multi-Core Process Clustering (Free, Config)
        ↓
3. Algorithmic & Database Index Fixes (Free, Code/Schema)
        ↓
4. In-Memory Caching & Write-Buffering (Moderate Cost, Infra/Code)
        ↓
5. Native Hot-Path Optimization (High Effort, Code)
        ↓
6. Horizontal Scale-Out & Edge Distribution (High Cost, Infra)
```

---

## 1. Framework & Runtime Tuning

Before changing infrastructure or architecture, optimize Solarch Server route handling overhead:
- **Middleware Streamlining**: Remove unnecessary logging, tracing, or body-parsing middleware from hot API paths.
- **Fast Serialization**: Use optimized JSON serialization for canonical API models instead of generic object dumping.
- **Payload Truncation**: Return only requested collection fields (`?fields=id,name`) rather than entire database records.

---

## 2. Multi-Core Process Clustering

A standard Node.js server process runs on a single event loop core. If host CPU shows idle capacity while throughput is plateaued:
- Enable Node.js `cluster` module or process manager (PM2/container workers) with worker count equal to available CPU cores ($N$).
- On a 4-core machine, this typically yields a 2.5x–3.8x throughput increase on CPU-bound routes at zero infrastructure cost.
- **Limit**: At very high core counts, reverse-proxy or parent-process connection dispatching can become a ceiling.

---

## 3. Algorithmic & Database Index Fixes

Algorithmic optimizations frequently yield 10x–1000x gains that dwarf any hardware upgrade:
- **Indexes on Collection Rules & Filters**: Ensure all fields queried in collection rules or public filters have matching database indexes in SQLite or PostgreSQL.
- **Cursor-Based Pagination**: Replace deep `OFFSET` pagination (which scans and discards $N$ rows) with cursor pagination (`WHERE id > :last_id LIMIT :limit`).
- **Batching & Eager Loading**: Replace loops of individual queries with batch queries (`WHERE id IN (...)`) to eliminate N+1 latency.

---

## 4. In-Memory Caching & Write-Buffering

When the database layer (`DatabaseDriver`) is the confirmed bottleneck and queries cannot be further indexed:
- **Read Caching**: Cache canonical JSON responses for idempotent read endpoints in an in-memory store (e.g. Redis, Valkey, or local memory cache with TTL).
- **Write-Buffering**: For high-frequency writes (e.g. telemetry, event logs, access tracking), buffer records in memory and flush in batched transactions to PostgreSQL/SQLite via background workers.
  - *Tradeoff*: Increases throughput dramatically while introducing an explicit durability window if uncommitted data in memory restarts before flush.
- **Sharding Beyond Single-Node In-Memory Ceilings**: A single Redis node caps out around ~100k ops/sec. Beyond this, shard across multiple nodes using key hashing. Use wide random IDs (e.g. UUIDv4 / 128-bit) to eliminate central lock contention.

---

## 5. Native Hot-Path Optimization

Only justified when profiling confirms that **CPU-bound computation within Solarch Server** (e.g., intensive serialization, cryptographic operations, or schema validation) remains the primary bottleneck after multi-core clustering and database optimizations:
- Offload specific hot operations to worker threads or native addons (e.g., Rust via NAPI-RS or WebAssembly).
- Keep the boundary minimal: retain Solarch Server for authentication, routing, and collection rules, delegating only the compute-intensive algorithm.
- *Caution*: Rewriting introduces substantial maintenance overhead. Never rewrite an entire service when only a single endpoint is bottlenecked.

---

## 6. Horizontal Scale-Out & Edge Distribution

When a single host is fully saturated across all cores, network, and memory:
- Deploy multiple Solarch Server container instances behind a managed load balancer (Fly.io machines, Render Web Services, or Kubernetes).
- Ensure Solarch Server statelessness: session authentication must use stateless tokens (JWT/signed tokens) or shared token stores; file storage must use persistent object storage rather than local disk.
- Verify that load balancer ingress and connection limits are provisioned for peak target RPS.
