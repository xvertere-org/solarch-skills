# Diagnosing Solarch API Bottlenecks

Never guess the bottleneck. Work through this checklist sequentially whenever Solarch Server API throughput or response latency degrades under load. Stop as soon as a tier is saturated—that is the active bottleneck.

---

## 1. Is the Load Generator Saturated?

Before inspecting Solarch Server, verify the benchmarking client:
- Check client CPU and network bandwidth during the test.
- If client CPU is pegged (>85%) or client network is saturated, the test results understate server capacity.
- **Action**: Increase worker threads (`-w`), scale client instances, or run distributed load generators before concluding the Solarch API is at capacity.

---

## 2. Is Solarch Server CPU Saturated?

Measure Solarch Server process CPU and aggregate host CPU:
- **Per-Core Pinning**: If one core is at 100% while other cores remain idle, Solarch Server is executing single-threaded request dispatch without multi-process clustering.
  - *Symptom*: Moderate aggregate CPU (e.g., 25% on 4 cores), high latency, flat RPS ceiling.
- **Full Host Saturation**: If all cores are near 0% idle across the host, the server is compute-bound.
  - *Symptom*: High CPU in Node runtime/V8 garbage collection, JSON serialization, or cryptographic hashing (auth tokens/passwords).

---

## 3. Is Network Bandwidth Saturated?

Calculate network throughput: $\text{bytes/sec} = \frac{\text{total bytes transferred}}{\text{test duration}}$.
- Compare with server network interface rating (convert Gbps to GB/s by dividing by 8).
- Solarch endpoints returning large collection payloads (many records or unprojected fields) frequently saturate the network long before CPU or database limits are reached.
- **Check**: Are responses returning unneeded columns or uncompressed payloads?

---

## 4. Is the Database Layer Saturated?

Solarch Server relies on the `DatabaseDriver` abstraction layer (SQLite, PostgreSQL, Neon PostgreSQL).
- **IOPS Limits**: Check storage IOPS and disk write queues on the database instance. High read/write IOPS with moderate DB CPU indicates disk I/O bottlenecks.
- **Database CPU**: Saturated database CPU typically indicates missing indexes or unoptimized query plans rather than hardware limits.
- **Connection Pool Exhaustion**: If requests wait in queue while server and DB CPU remain low, the driver connection pool size is too small or leaking open connections.

---

## 5. Is it an Algorithmic / Query Mistake?

Before scaling hardware or adding caching tiers, verify query complexity in route handlers:
- **Unindexed Filtering**: Collection queries filtering on fields without database indexes force full table scans ($O(n)$).
- **Full Collection Counts**: Running `SELECT COUNT(*)` across large collections on every read path.
- **N+1 Record Hydration**: Looping over records to execute secondary queries (e.g. fetching author profiles or relations individually).
- **Inefficient Random Ordering**: `ORDER BY RANDOM()` on large tables.
- **Diagnostic Tell**: App server CPU is low, database CPU is moderate, network is idle, but p99 latency spikes into seconds.

---

## 6. Is an Upstream Gateway or Proxy Saturated?

When Solarch Server is deployed behind a reverse proxy, load balancer, or edge network (Fly.io, Render, Cloudflare Workers):
- Check proxy connection limits, TLS handshake rate limits, or HTTP keep-alive timeouts.
- Compare direct-to-server benchmark numbers against gateway-fronted benchmark numbers. If direct throughput is substantially higher, the bottleneck is upstream configuration.

---

## Diagnosis Reporting Standards

Every performance diagnosis must report:
1. **Identified Bottleneck Layer**: Specific component (e.g., "PostgreSQL IOPS", "Solarch Server single-core CPU pinning", "Client generator saturation").
2. **Empirical Evidence**: Exact metric with units (e.g., "Database disk write queue length: 24, IOPS pegged at 3,000/3,000; Server CPU was at 14%").
3. **Eliminated Layers**: Explicit list of layers checked and ruled out (e.g., "Network utilization at 12% of 10 Gbps cap; Load generator client CPU at 28%").
