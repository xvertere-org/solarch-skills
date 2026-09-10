# Benchmarking Solarch API Routes & Server

## 1. Tooling & Setup

For benchmarking Solarch Server routes and endpoints, use a dedicated HTTP/WebSocket load generator rather than manual testing or simple shell loops.

- **`autocannon`** (Node.js, cross-platform default):
  ```bash
  autocannon -c <connections> -d <duration_seconds> -p <pipelining> -m <method> -b <json_body> -H "Content-Type: application/json" <url>
  ```
- **`wrk`** or **`k6`**: Recommended for high-concurrency or scripted multi-stage scenario tests.

### Essential Flags & Concurrency Math

- `-c <connections>`: Concurrent TCP connections established to the Solarch Server.
- `-d <duration>`: Duration in seconds. Use short runs (10–30s) during rapid iteration; use sustained runs (10–30 min) to detect connection-pool exhaustion, garbage collection pauses, or database driver connection leaks.
- `-p <pipelining>`: Back-to-back requests on each connection before reading response.
- `-w <workers>`: Worker threads spawned on the load generator client machine. Without adequate client workers, the benchmark tool itself becomes the bottleneck and reports false server limits.

$$\text{Instantaneous In-Flight Requests} \approx \text{connections} \times \text{pipelining}$$

Always check the **error count** alongside average requests/second. A high RPS with non-zero 5xx or connection resets indicates saturated or failing endpoints, not viable capacity.

---

## 2. Solarch Component Isolation

When benchmarking, isolate which tier is being exercised:

1. **Client / Core-Client Workloads**: If measuring `@solarch/core-client` query building or serialization, run localized micro-benchmarks without network overhead.
2. **Solarch Server API Routes**: Point load generator directly at server endpoints (e.g. `/api/records`, `/api/auth/token`).
3. **Database Driver Layer**: Test with real `DatabaseDriver` backends (SQLite, PostgreSQL, Neon PostgreSQL) to separate route handling from storage I/O.
4. **Deployment / Edge Infrastructure**: When testing deployed apps (Fly.io, Render, Cloudflare Workers), test behind and direct-to-origin to detect load-balancer/proxy caps.

---

## 3. CPU and Core Utilization Math

Always specify the measurement convention when reporting CPU metrics:

- **Single-Core Utilization**:
  $$\text{Core Utilization} = \frac{\text{Total Time} - \text{Idle Time}}{\text{Total Time}} \times 100\%$$
- **Method 1 (Process / Core Sum)**: Each fully utilized core adds 100%. A fully saturated 8-core machine reports 800%. A single-threaded Node.js server process pegging one core reports 100%.
- **Method 2 (Normalized)**: Sum divided by core count, capped at 100%. The same fully saturated 8-core machine reports 100%.

> **The Single-Thread Pinning Tell**: If Solarch Server process shows ~100% CPU on Method 1 while the system shows high idle percentage on a multi-core machine, the server is single-thread bound. The immediate remedy is process clustering or worker pooling, not a larger machine.

---

## 4. Network Bandwidth Calculation

Cloud providers quote network interfaces in **bits per second** (e.g., 10 Gbps, 25 Gbps), while load tools report in **bytes per second**.

$$\text{Rated Throughput (GB/s)} = \frac{\text{Rated Speed (Gbps)}}{8}$$

Example: Measured 1.1 GB/s on a 10 Gbps (~1.25 GB/s) interface means the network is at ~88% capacity and nearing saturation. If response bodies are large JSON collections, payload trimming or compression yields higher RPS gains than compute scaling.

---

## 5. Iteration Hygiene

- **Single Variable Rule**: Change only one variable at a time (e.g., enable Node cluster mode, OR add an index, OR enable response caching). Never combine fixes between runs.
- **Client Headroom Verification**: Verify that client machine CPU is <70% during tests to ensure load generation is not artificially capping throughput.
- **Warm-Up Runs**: Execute an initial 10-second warm-up run before recording baseline figures to allow JIT compilation, connection pooling, and connection handshakes to stabilize.
