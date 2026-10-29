# Technical Architecture

**Status:** MVP **tech stack selected** 28 October 2026 (DEC-033); system architecture design **DRAFT / NOT FROZEN**.

## Selected technology

- [Tech Stack Decision v1.0](tech-stack-decision-v1.0.md) — **accepted technology baseline**: React/Vite/TypeScript, Tailwind/shadcn/ui, Hono + Cloudflare Workers Static Assets, D1 (SQLite), Drizzle ORM, Cloudflare Access OTP + backend RBAC, private R2, Zod, Vitest, Playwright, GitHub Actions and Wrangler.
- Browser Run HTML-to-PDF is the preferred renderer, **subject to a successful staging proof-of-concept**; it is not a production-verified implementation.
- **Hostinger shared/VPS and PostgreSQL** were considered but are **not the selected MVP stack**. No dual database.
- Modular monolith; no unnecessary microservices, Kubernetes, Redis or extra queues.

## Database schema design — v0.2 corrected, ready for review

- **[Database ERD v0.2](erd-database-v0.2.md)** — corrected 15-table Mermaid ERD; nine prioritized v0.1 audit findings addressed in design; 20 unexecuted DB test cases.
- **[Database Data Dictionary v0.2](data-dictionary-v0.2.md)** — changed/new fields, composite FK consistency, policy selection, source hashing, Attribute blind state, restricted cell audit, signer-owner binding, concurrency-safe report finalization.
- [Mermaid source v0.2](erd-v0.2.mmd) — editable diagram.
- Archived baseline: [ERD v0.1](erd-database-v0.1.md), [Data Dictionary v0.1](data-dictionary-v0.1.md) and [Mermaid v0.1](erd-v0.1.mmd).

**Status:** v0.2 corrections are **DESIGN ONLY / DRAFT**; not executed D1 migrations or deployed database. Cross-revision foreign keys, immutable finalize triggers, D1/R2 reconciliation, quality criteria and retention still require implementation proof.

## Detailed design under review

- [System Architecture Design v0.1](system-architecture-v0.1.md) — proposed entities, RBAC, input versioning, pure calculation module, Supervisor approval, D1/R2 report reconciliation, API endpoints, operations and backup.
- Access OTP must replace the historical S01 password screen in a later reviewed Field & Screen revision.
- Ratama Quality acceptance thresholds, ERD/schema, security and staging PDF tests, authentic signature assets, production cost/restore tests, and high-fidelity visual polish are not yet approved.

**Decision boundary:** Stack direction accepted; **not** all architecture, calculation criteria, product spec or production readiness. No code or infrastructure has yet been deployed.
