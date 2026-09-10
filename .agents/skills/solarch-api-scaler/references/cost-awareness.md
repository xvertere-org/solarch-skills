# Cost-Aware Scaling for Solarch Deployments

At high throughput, infrastructure costs scale rapidly. Every scaling recommendation must attach an estimated monetary cost alongside the projected performance gain.

---

## 1. Cost Delta per Scaling Tier

Never propose infrastructure upgrades without calculating the approximate monthly cost increase:

| Scaling Action | Typical Cloud Cost Impact | Verification Prerequisite |
| :--- | :--- | :--- |
| **Enable Node.js Clustering** | **$0 / month** | Confirm multi-core CPU capacity exists. |
| **Add Database Indexes** | **$0 / month** (negligible disk) | Profile slow queries and missing index scans. |
| **Increase Provisioned DB IOPS** | **+$50 – $1,000+ / month** | Confirm disk write queues / IOPS saturation before upgrading. |
| **Scale Up Database Instance** | **+$100 – $5,000+ / month** | Confirm DB CPU/memory saturation, not connection pooling flaws. |
| **Add In-Memory Cache (Redis/Valkey)** | **+$30 – $500+ / month** | Confirm database read queries are cacheable and repetitive. |
| **Add Solarch Server Replica Nodes** | **+$20 – $200 / node / month** | Confirm app CPU is saturated across existing instances. |

When multiple small upgrades accumulate to thousands of dollars per month, evaluate algorithmic fixes or caching before adding more hardware.

---

## 2. The Cost-Per-Million-Requests Sanity Check

For Solarch deployments utilizing per-request or serverless pricing (e.g., Cloudflare Workers, AWS Lambda, managed API gateways):

$$\text{Monthly Cost} = \text{Requests/Sec} \times 2,592,000 \times \text{Price per Request}$$

*(Note: There are approximately 2,592,000 seconds in a 30-day month).*

### Sanity Check Example
- At **20,000 RPS sustained**:
  - Total requests/month: $\approx 51.8 \text{ billion requests}$.
  - On a per-request model billed at \$0.50 per million: **\$25,920 / month**.
  - On dedicated compute (e.g., clustered Solarch Server instances on Fly.io/Render with a PostgreSQL instance): **\$200 – \$800 / month**.

> **Rule**: When sustained throughput exceeds several thousand requests per second, dedicated capacity-billed infrastructure is drastically cheaper than per-request serverless billing.

---

## 3. Load Testing Budgeting & Safeguards

Benchmarking high-throughput systems generates significant egress and compute costs:
- **Client Egress Costs**: Generating 50k RPS with 2 KB payloads transfers ~100 MB/s ($\approx 360 \text{ GB/hour}$). In cloud environments, cross-zone or internet egress is billable.
- **Time-Box Tests**: Keep load tests strictly bounded (e.g., 60 seconds for micro-tests, 15 minutes for endurance tests). Never run unbounded load tests in CI/CD pipelines.
- **Run Load Generators Inside the Same Cloud Region**: Co-locate benchmark client instances in the same cloud region/VPC to eliminate internet egress charges and avoid measuring internet transit latency.
