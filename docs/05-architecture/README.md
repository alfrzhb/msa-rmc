# Technical Architecture

**Status:** System Architecture v0.1 recommended for stakeholder review — DRAFT / NOT FROZEN (28 October 2026)

## Primary review document

- [System Architecture Design v0.1](system-architecture-v0.1.md) — proposed **TypeScript modular monolith**: React/Vite SPA and Hono API on a single Cloudflare Worker, Access OTP, D1, private R2 and Browser Run PDF.
- Covers relational schema, MSA calculation boundaries, Supervisor approval, Known Reference custody, immutable revision/report storage, 14 sample REST endpoints, security and operations.

## Explicit review gates

- The proposed stack is **not yet product-owner approved** and no application code or infrastructure was deployed.
- Access OTP replaces the older S01 username/password concept only after the screen specification is reconciled.
- Ratama Quality still must approve policy risk/acceptance criteria and exception handling.
- Staging PDF rendering, version-guarded D1 finalization, R2 archival, recovery, and true JWT/RBAC tests require proof-of-concept verification.
- Logo/signature assets and high-fidelity color refresh are not final.

Single-client MVP first. Avoid microservices, unnecessary queues, and premature multi-tenancy; retain modular source boundaries.
