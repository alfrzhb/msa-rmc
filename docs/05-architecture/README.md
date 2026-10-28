# Technical Architecture

**Status:** MVP **tech stack selected** 28 October 2026 (DEC-033); system architecture design **DRAFT / NOT FROZEN**.

## Selected technology

- [Tech Stack Decision v1.0](tech-stack-decision-v1.0.md) — **accepted technology baseline**: React/Vite/TypeScript, Tailwind/shadcn/ui, Hono + Cloudflare Workers Static Assets, D1 (SQLite), Drizzle ORM, Cloudflare Access OTP + backend RBAC, private R2, Zod, Vitest, Playwright, GitHub Actions and Wrangler.
- Browser Run HTML-to-PDF is the preferred renderer, **subject to a successful staging proof-of-concept**; it is not a production-verified implementation.
- **Hostinger shared/VPS and PostgreSQL** were considered but are **not the selected MVP stack**. No dual database.
- Modular monolith; no unnecessary microservices, Kubernetes, Redis or extra queues.

## Detailed design under review

- [System Architecture Design v0.1](system-architecture-v0.1.md) — proposed entities, RBAC, input versioning, pure calculation module, Supervisor approval, D1/R2 report reconciliation, API endpoints, operations and backup.
- Access OTP must replace the historical S01 password screen in a later reviewed Field & Screen revision.
- Ratama Quality acceptance thresholds, ERD/schema, security and staging PDF tests, authentic signature assets, production cost/restore tests, and high-fidelity visual polish are not yet approved.

**Decision boundary:** Stack direction accepted; **not** all architecture, calculation criteria, product spec or production readiness. No code or infrastructure has yet been deployed.
