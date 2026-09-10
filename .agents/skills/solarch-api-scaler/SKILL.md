---
name: solarch-api-scaler
description: Diagnose and scale throughput bottlenecks for Solarch APIs, endpoints, and server instances under heavy load (thousands to millions of requests per second). Use when benchmarking API capacity, diagnosing route latency degradation under load, scaling Solarch Server, analyzing CPU/core utilization, optimizing database throughput on SQLite or PostgreSQL, adding Redis caching or write buffers, or evaluating cloud infrastructure cost at scale. Trigger on queries like "How many requests per second can this Solarch API handle?", "Our Solarch endpoint falls over at 20k RPS", "Find the bottleneck in this API", "Should we add Redis?", "Should we move this hot path to Rust?", "Benchmark this Solarch route", "Our database is maxed out under load", or "How should we scale this Solarch backend?". Do NOT trigger on normal CRUD tasks, simple code cleanup, or styling.
---

# Solarch API Scaler

## Purpose

Diagnose, benchmark, and eliminate throughput and latency bottlenecks in Solarch Server endpoints and APIs under concurrent load. Determine the true saturated layer before proposing code, database, or infrastructure changes, adhering to the discipline:

$$\text{MEASURE} \longrightarrow \text{IDENTIFY BOTTLENECK} \longrightarrow \text{CHEAPEST EFFECTIVE FIX} \longrightarrow \text{RE-BENCHMARK}$$

---

## References

### Ecosystem Foundations
- [Solarch Architecture & Component Boundaries](../references/solarch-architecture.md)
- [Solarch Engineering Principles](../references/engineering-principles.md)

### Scaler Methodologies
- [Benchmarking Methodology & Tooling](references/benchmarking.md)
- [Bottleneck Diagnosis Elimination Checklist](references/bottleneck-diagnosis.md)
- [Scaling Strategies (Cheapest to Most Expensive)](references/scaling-strategies.md)
- [Cost Awareness & Infrastructure Sanity Checks](references/cost-awareness.md)

---

## Solarch Architectural Boundaries

Always identify the exact component layer involved in performance investigations:

1. **Solarch Server**:
   - Manages REST/WebSocket endpoints, authentication, authorization collection rules, and realtime event dispatching.
   - Executes the single-threaded Node.js event loop by default. Requires multi-process clustering to utilize multi-core host compute.
2. **Database Driver Layer (`DatabaseDriver`)**:
   - Manages connections to SQLite, PostgreSQL, and Neon PostgreSQL.
   - Saturated database IOPS or CPU must be resolved with indexes or query tuning before scaling database tiers.
3. **`@solarch/core-client`**:
   - Strictly platform-independent client protocol layer. Does not run server-side database drivers. Client-side query serialization bottlenecks belong in client profiling, not server scaling.
4. **Deployment Targets**:
   - Fly.io, Render Web Services, Cloudflare Workers. Load-balancer and reverse-proxy ceilings must be checked independently of backend server capacity.

---

## Core 5-Stage Scaling Workflow

### Stage 1: Baseline & Instrument First
- Establish a reproducible baseline using an HTTP load generator (`autocannon`, `wrk`, or `k6`) before editing any code.
- Collect simultaneous metrics across: client load generator CPU, Solarch Server host CPU (per-core and aggregate), network interface throughput, and database IOPS/CPU.
- Follow [Benchmarking Methodology](references/benchmarking.md) to ensure the client is not capping results.

### Stage 2: Walk the Elimination Checklist
- Step sequentially through the checklist in [Bottleneck Diagnosis](references/bottleneck-diagnosis.md):
  1. *Client load generator saturation?*
  2. *Server CPU core pinning (single-threaded process bound to 1 core)?*
  3. *Network bandwidth ceiling (Gbps vs GB/s)?*
  4. *Database driver IOPS or CPU limits?*
  5. *Algorithmic or unindexed query flaws ($O(n)$ collection scans, N+1 queries)?*
  6. *Upstream gateway or reverse proxy connection limits?*
- Never assume "the database is the bottleneck" or "Node is slow" without empirical metric proof.

### Stage 3: Apply the Cheapest Effective Fix
- Work strictly through the hierarchy in [Scaling Strategies](references/scaling-strategies.md):
  1. **Framework & serialization tuning** (free, code).
  2. **Multi-core process clustering** (free, config).
  3. **Algorithmic query & index fixes** (free, schema).
  4. **In-memory cache tier (Redis/Valkey) or write-buffering** (moderate cost).
  5. **Native worker thread or compiled hot-path optimization** (high effort).
  6. **Horizontal multi-instance scale-out** (high cost).

### Stage 4: Verify Cost Feasibility
- Apply the sanity checks in [Cost Awareness](references/cost-awareness.md):
  - State the expected $\$ / \text{month}$ delta for any infrastructure modification.
  - Run the cost-per-million-requests calculation to verify that serverless/per-call billing is not financially unviable compared to dedicated instances at high sustained RPS.

### Stage 5: Re-Benchmark Isolated Changes
- Apply only **one change at a time**.
- Re-run the baseline benchmark with identical parameters.
- Verify error counts remain zero and record the new binding constraint.

---

## Guardrails: Rejecting Premature Optimization

1. **Low-Traffic Requests**: If traffic is modest (e.g., 50–500 RPS), reject requests to rewrite handlers in Rust/C++, add Redis clusters, or shard databases. Standard Solarch Server and PostgreSQL easily handle these loads without complex infrastructure.
2. **Missing Evidence**: Never recommend external cache tiers (Redis) or hardware upgrades until profiling proves the database or compute layer is saturated.
3. **Preserve Safety**: Never bypass collection rule authorization, input validation, or data persistence guarantees to artificially inflate benchmark numbers.

---

## Cross-Skill Boundaries

- **`solarch-api-scaler`**: Owns throughput measurement, saturation diagnosis, load testing, and scaling decisions.
- **`solarch-code-quality`**: Owns readability, type safety, and internal code architecture. (Triggered when code structure needs review, not capacity scaling).
- **`solarch-refactor-review`**: Owns structural refactoring proposals and abstraction viability.
- **`solarch-testing`**: Owns unit, integration, and contract test suites.
- **`solarch-deployment`**: Owns deploying the optimized application to cloud providers.

---

## Output Standard

When delivering performance assessments, provide:
1. **Measured Throughput**: Baseline RPS, latency percentiles (p50, p95, p99), and error count.
2. **Binding Constraint**: Exact component and metric proving saturation (e.g., "PostgreSQL storage IOPS capped at 3,000; Solarch Server host CPU idle at 82%").
3. **Eliminated Components**: Tiers verified and ruled out.
4. **Recommended Next Step**: The cheapest effective fix and its estimated implementation effort and infrastructure cost delta.
