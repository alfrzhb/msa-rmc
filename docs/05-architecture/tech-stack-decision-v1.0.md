# RMC MSA — Tech Stack Decision v1.0

**Decision:** SELECTED FOR MVP — PRODUCT-OWNER ACCEPTED  
**Date:** 28 October 2026  
**Scope:** Technology selection for the single-client Ratama MSA MVP. **Not** a freeze of all architecture details, statistical formulas, Quality criteria, or deployment readiness.

## 1. Selected technology baseline

| Layer | Technology | Status / intended use |
| --- | --- | --- |
| Language | **TypeScript** | Common type-safe language across web, API, schemas and calculation engine |
| Frontend | **React + Vite** | Responsive SPA; desktop and mobile UX follow the approved wireframe |
| UI | **Tailwind CSS + shadcn/ui** | Reusable accessible components; visual polish v0.2 still pending |
| API | **Hono + Cloudflare Workers** | Modular monolith, one API runtime, no VPS to administer |
| Static hosting | **Cloudflare Workers Static Assets** | SPA and API under the Cloudflare deployment model |
| Database | **Cloudflare D1 (SQLite)** | Primary relational database for MVP; SQLite is the D1 engine |
| ORM / migrations | **Drizzle ORM + reviewed migrations** | Typed query and explicit schema evolution |
| Authentication | **Cloudflare Access (email OTP)** | Account identity / access gate, subject to JWT integration tests |
| Authorization | **Server-enforced RBAC** | Admin, Supervisor, Operator; per-study ownership, blinded references, revision state |
| Private file storage | **Cloudflare R2** | Final PDF files, company logo and authorized signatory asset |
| PDF reports | **HTML/CSS print template + Cloudflare Browser Run** | Preferred approach, **technical proof-of-concept required** before locking the renderer |
| Request/data validation | **Zod** | API and field validation, not a substitute for server-side permission checks |
| Automated testing | **Vitest + Playwright** | Golden calculation regression, component/integration and end-to-end tests |
| CI/CD | **GitHub Actions + Wrangler** | Checks and staging/production deployment; credentials via secret management |
| Code architecture | **TypeScript modular monolith** | Separate studies, formulas, policies, approval, reports and audit modules |

### Not selected for the initial MVP

- **Hostinger shared hosting / VPS:** discussed and **not chosen** for the MVP deployment. The primary hosting provider is Cloudflare.
- **PostgreSQL + Neon/Hyperdrive:** feasible alternative for later database requirements, **not** the MVP database. No dual-write/dual-database design.
- **Microservices, Kubernetes, Redis, background queues:** unnecessary until a demonstrated operational requirement exists.
- **Full XLSX file import:** deferred; manual entry and paste-from-Excel only for MVP.
- **Cloudflare Pages as an additional frontend deployment:** not required under the one-Worker-with-static-assets baseline.

## 2. Why Cloudflare + D1 was chosen

For **one Ratama client** and a limited number of users, managed Workers, D1 and R2 remove shared-hosting compatibility issues and VPS maintenance while keeping MSA source data, approval and audit records relational. D1 supports SQLite—not native PostgreSQL. A future move to managed PostgreSQL is possible but would require **schema/query/migration work**, and is not a zero-change toggle.

The choice does **not** assert that D1 is always superior to PostgreSQL. Review D1 quotas, database size, query latency, connection/concurrency behavior and paid-plan costs against actual usage before production. Select a production Cloudflare plan only after cost and operational testing, rather than assuming Free is production-suitable.

## 3. Architecture implementation contract

~~~text
Browser
  React + Vite SPA (Workers Static Assets)
      |
Cloudflare Access (email OTP; verified JWT)
      |
Hono API — Cloudflare Workers (RBAC, Zod)
      |
      +-- Studies / Draft / History / Revision
      +-- Known Reference (Supervisor-only)
      +-- Pure TypeScript MSA Calculation Engine
      +-- Quality Judgment Policy (versioned)
      +-- Submit / Supervisor Approve & Finalize
      +-- Audit + Report Generation
      |
      +-- D1 (SQLite): relational data, revisions, results, audit, approvals
      +-- R2 (private): immutable final PDF, logo and signature assets
~~~

**MSA safeguards that must survive implementation:**

- **Variable:** 10 parts × 3 appraisers × 3 trials = 90 readings, initial validated Average & Range method.
- **Attribute:** 50 samples × 3 appraisers × 2 trials = 300 decisions plus 50 *separate* Known References; pooled per-appraiser Kappa method is the agreed design direction.
- **Operator:** can view all study History, edit only own Drafts, submit for approval; no access to active blinded Known References.
- **Supervisor:** alone manages Known Reference, reviews submissions and **Approves & Finalizes**; finalized revisions must not be edited in place.
- **Admin:** manages app accounts and roles, **not automatically** a study Supervisor.
- Calculate on the **backend** with versioned, independently verified formulas and golden datasets; avoid falsely treating workbook arithmetic defects as correct.
- Source changes invalidate calculated results and submitted reviews; missing Quality-approved acceptance criteria yield **Review Required**, never invented OK.
- Generate reports from the exact approved snapshot; use private R2 keys and versioned audit/approval metadata, not a shared mutable file. D1 and R2 are not one transaction: design idempotent, concurrency-guarded finalization plus recovery for partial failures.

## 4. Reconciliation and remaining proof gates

1. **Login UX:** Earlier Field & Screen v0.3 depicts an email/password form. With Access OTP, replace that in a future UI spec revision with **Continue with work email → Cloudflare Access OTP → authenticated app**. No custom password store.
2. **Real Access security:** Validate JWT signature, audience, issuer, expiry and active user mapping; enforce authorization on the Worker. Cross-role UI switching in prototypes is demonstration only.
3. **D1 schema:** ERD, unique constraints for each reading/judgment slot, migrations, optimistic concurrency, recovery and indexes must be reviewed.
4. **PDF proof:** Confirm Browser Run HTML-to-PDF support, fonts, print pagination, costs, quotas and error handling in staging. R2 stores real reports only after valid Supervisor approval; do not fabricate signatures.
5. **Quality approval:** Ratama/customer must still approve final Attribute NG-miss/false-reject policy and any exceptional disposition before automated "OK".
6. **Operations:** Staging/production isolation, D1 backup and tested restore, R2 retention, role offboarding, event logs, monitored failures and costs.
7. **UI:** High-Fidelity concept accepted provisionally, but navy/teal-heavy palette needs the documented Visual Polish v0.2.

## 5. Scope and change control

This decision **locks the selected stack direction for design/implementation planning**, but does **not** freeze the overall product requirements, API schema, ERD, deployment plan, mathematical methodology or release readiness. Tech changes require an explicit new decision entry, with rationale and migration impact.

See [System Architecture Design v0.1](system-architecture-v0.1.md), [Field & Screen Specification v0.3](../04-ui-ux/field-and-screen-specification-v0.3.md), [Calculation Decisions v0.3](../03-calculations/calculation-decision-review-v0.3.md) and the [Decision Log](../decisions/decision-log.md).
