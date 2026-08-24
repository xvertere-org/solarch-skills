---
name: solarch-security-review
description: Conduct a Solarch-focused security audit on code, APIs, authentication, authorization, collection rules, secrets, and deployment configs. Use when evaluating security risks, auditing authentication or authorization changes, reviewing PRs for security problems, or checking data safety in Solarch. Trigger on queries like "security audit", "check for vulnerabilities", "is this secure?", "review auth", "review for security problems", or "audit this API".
---

# Solarch Security Review

## Purpose

Perform rigorous security audits on Solarch components, APIs, SDKs, and configurations to detect security vulnerabilities, secret leakage, authorization flaws, and trust-boundary violations.

---

## Shared References

- [Solarch Architecture & Boundaries](../references/solarch-architecture.md)
- [Solarch Engineering Principles](../references/engineering-principles.md)

---

## Audit Focus Areas

### 1. Authentication & Session Security
- Token storage, transmission, and lifecycle management (access/refresh tokens).
- Revocation, logout cleanup, state machine transitions, credential handling.

### 2. Authorization & Server Boundaries
- **Server Enforcement**: Verify collection rules, RBAC, and record access are enforced **on the server**. Client-side checks are UI conveniences only.
- Privilege escalation vectors, Admin endpoint access controls, multi-tenant boundaries.

### 3. Input Security & Validation
- Validation of all request parameters, headers, file uploads, and JSON payloads.
- SQL/Database injection, unsafe query construction, path traversal, unsafe deserialization.

### 4. Secret & Credentials Management
- Detect hardcoded API keys, JWT secrets, database connection strings, or environment tokens in source code, logs, or error responses.

### 5. Web & Transport Security
- XSS, CSRF tokens, CORS configurations, secure/HTTP-only cookie flags, SSR safety, redirect validation.

### 6. Solarch-Specific Security Vectors
- **Realtime Channel Auth**: Verify clients cannot subscribe to unauthorized record change feeds.
- **File Storage Access**: Ensure file access rules mirror record authorization rules.
- **Webhook & Event Integrity**: Verify signature checks on external webhook receivers.

---

## Severity Classification

Report findings using these precise categories:

- **Confirmed Vulnerability**: Exploitable defect breaking security guarantees (e.g., missing collection rule check on server endpoint).
- **Likely Risk**: Defect that increases attack surface or relies on fragile assumptions (e.g., unhandled token refresh failure leaking raw error trace).
- **Defense-in-Depth Recommendation**: Architectural hardening suggestion (e.g., adding explicit `SameSite` flags to cookies).

*Rule*: Do not fabricate vulnerabilities or flag standard coding patterns as security risks without concrete evidence of risk.
