# RMC MSA — System Architecture Design v0.1

**Status:** TECHNOLOGY STACK SELECTED (DEC-033); ARCHITECTURE DETAILS UNDER TECHNICAL REVIEW — DRAFT / NOT FROZEN  
**Date:** 28 October 2026  
**Scope:** Single-client MVP; design and decision preparation only, **no application implementation**.

**Stack decision:** [Tech Stack Decision v1.0](tech-stack-decision-v1.0.md) is now **product-owner accepted** (DEC-033). The selected services/frameworks in this document are no longer merely a hosting proposal; **database schema, API contracts, Access login reconciliation, PDF renderer feasibility, infrastructure details, statistical policies and release gates remain DRAFT**.

## 1. Recommendation

Use a **TypeScript modular monolith** deployed as **one Cloudflare Worker with static SPA assets and an Hono API**, backed by **Cloudflare D1 (SQLite)** and a **private R2 bucket**. Authentication is handled by **Cloudflare Access (email OTP)**, and study-level RBAC is enforced independently in the Hono backend. Render PDFs from an approved, versioned HTML report using **Cloudflare Browser Run /pdf** (validate template rendering before committing to production).

This architecture minimizes infrastructure while preserving testable boundaries among studies, MSA formulas, policies, approval, reports and audit. **Microservices, Kubernetes, Redis and queues are not part of MVP**.

## 2. Baseline requirements

- Admin manages application users; Supervisor exclusively manages Attribute Known Reference and **Approves & Finalizes**; Operator creates/edits only own Draft, but sees all-study History (with blinded-source restrictions).
- Variable MVP: 10 parts × 3 appraisers × 3 trials = **90 readings**, Crossed Average & Range.
- Attribute MVP: 50 samples × 3 appraisers × 2 trials = **300 decisions**, plus **50 separate verified Known Reference labels** controlled by Supervisor.
- Attribute's primary method: **Ratama pooled Kappa vs Known Reference per appraiser**; never silently substitute Minitab trial-average Kappa or compute a new global average.
- No automatic Quality OK/NG acceptance without customer/Quality-approved, versioned policy; undefined metrics and warnings are explicit.
- Partial Draft save; Current/Stale calculations; single-level Supervisor approval; immutable Final revisions and report PDF; revisions create new records.
- Responsive desktop and mobile UI direction approved (DEC-031); high-fidelity concept provisionally accepted but **visual palette still pending** (DEC-032).
- Historical Ratama calculation source files are evidence, not correct-by-assumption formula code.

## 3. Selected technology

| Layer | Proposed choice | Decision reason |
| --- | --- | --- |
| Web UI | React + Vite + TypeScript; React Router, TanStack Query | Fast, well-suited to an internal responsive data-entry SPA |
| Visual components | Tailwind CSS + selective shadcn/ui; accessible custom spreadsheet grid | Reuse design-system parts without forcing an unsuitable table widget |
| API | Hono on Cloudflare Workers, TypeScript | Small typed REST API, one deployable runtime |
| Asset hosting | Cloudflare Workers Static Assets, same hostname as API | Simpler routing and avoids unnecessary cross-origin session handling |
| DB | Cloudflare D1 / SQLite with relational constraints | Single-client and small-workload fit; inspect limits before production |
| DB access | Drizzle ORM + checked-in SQL migrations | Type-safe schema and migration review; no runtime auto-migrate |
| Authentication | Cloudflare Access, permitted business email accounts / one-time PIN | No custom password storage or password reset lifecycle |
| App authorization | Server-side RBAC + owner checks in D1 | Access authentication **does not** grant Supervisor/Operator permissions |
| Files | Private R2 bucket, versioned report keys | PDF bytes, authorized signature and logo assets |
| PDF | Browser Run PDF Quick Action from server-generated HTML | Render report layout with print CSS, then archive exact bytes |
| Validation | Zod both boundary and API schemas | Explicit date/decimal/enum rules and typed contract |
| Tests | Vitest unit/integration, Playwright responsive/E2E | Independent golden calculation fixtures and role/approval regression |
| CI/CD | GitHub Actions + Wrangler, separate staging / production resources | Automated checks, controlled deployments, migration safety |

Primary references: [Workers SPA+API](https://developers.cloudflare.com/workers/static-assets/), [Hono on Workers](https://developers.cloudflare.com/workers/framework-guides/web-apps/more-web-frameworks/hono/), [D1 batch](https://developers.cloudflare.com/d1/worker-api/d1-database/), [Access JWT validation](https://developers.cloudflare.com/cloudflare-one/access-controls/applications/http-apps/authorization-cookie/validating-json/), [Browser Run PDF](https://developers.cloudflare.com/browser-run/how-to/pdf-generation/), [R2 bucket locks](https://developers.cloudflare.com/r2/buckets/bucket-locks/).

### 3.1 Authentication/UI reconciliation

The older S01 specification proposes an **email/password** login, whereas this architecture proposes Cloudflare Access's **hosted email OTP**. Do **not** build a password field when choosing Access. Update the UI specification in a future reviewed version: branded "Continue with work email" entry/redirect → Cloudflare Access OTP → app session. No public self-registration.

Cloudflare Access must protect **production and preview hostnames**, including API endpoints. The Worker must verify the Access JWT's signature, issuer, audience, expiry and subject, then map the verified identity to an **active** application user in D1 before deciding RBAC. The Admin account screen manages local app user/role/disabled state; provisioning Access allow rules is a separate administrative concern and must be documented to avoid inconsistent onboarding.

## 4. Component diagram

~~~text
Browser (React SPA / responsive Data Grid)
          |
  Cloudflare Access OTP
          |
  Cloudflare Worker (static assets + Hono /api/*)
          |
          +-- JWT verification + app-user lookup + RBAC
          +-- Studies / Draft Data Entry / History
          +-- Supervisor-only Known Reference
          +-- Pure Calculation Engine
          +-- Versioned Quality Policy & Results
          +-- Submit / Supervisor Approve & Finalize
          +-- Report HTML → Browser Run PDF → private R2
          +-- Audit
          |
          +-- D1: relational source data & snapshots
          +-- R2: PDFs, logo, authorized signature assets
~~~

The calculation module is **pure, deterministic TypeScript** and can be unit-tested without Cloudflare or a database.

## 5. Proposed relational model (companion ERD v0.1 prepared)

**Current detailed candidate: [Database ERD v0.2](erd-database-v0.2.md), [Data Dictionary v0.2](data-dictionary-v0.2.md), [Mermaid v0.2](erd-v0.2.mmd).** These supersede the earlier ERD/Dictionary v0.1 for technical review and specify selected_policy_id, composite review/approval FKs, server-calculated source digest, Attribute OPEN/LOCKED/DISCLOSED, restricted before/after audit and finalization triggers. This overview below is historical v0.1 and remains DRAFT; no D1 migrations have run.

| Table | Main responsibility | Important constraints |
| --- | --- | --- |
| users | Access subject, email, name, active flag, Admin/Supervisor/Operator role | Verified Access identity mapped to one active app user |
| studies | Stable study ID, type, creator, current revision, soft-delete flag | Study type immutable once source data exist |
| study_revisions | Draft/Finalized/Superseded revision, policy selection, input digest and blind-state lock | Finalized/Superseded immutable; one Draft per study |
| appraisers | Human identities who performed readings | Distinct within revision; **not** required to have login |
| samples | Part or sample IDs and order | Unique per revision, stable across appraisers |
| variable_readings | Revision, appraiser, sample, trial, decimal reading | Unique measurement slot, exactly 90 for validated preset |
| attribute_decisions | Revision, appraiser, sample, trial, OK/NG | Unique judgment slot, exactly 300 for validated preset |
| attribute_references | Revision, sample, known OK/NG, source, verifier | Unique sample reference; Supervisor-only service permissions |
| calculation_runs | Input hash, method version, result snapshot, warnings, calculated timestamp | Preserve reproducibility; invalidated when Draft changes |
| judgment_policies | Policy family, version, approval evidence, effective rules | Immutable approved version, no edits to formulas |
| review_requests | Submitted by/at, current review flag, correction reasons | Changes to input make existing review request stale |
| approvals | Supervisor identity, timestamp, disposition, policy snapshot, report linkage | Only authentic Supervisor action creates approval |
| report_artifacts | Content hash, R2 key, size, revision, release status | Unique key per immutable final report |
| audit_events | Actor, action, study revision, time, change context | Append-only from application; no normal UI deletion |
| brand_assets | Logo and signer-authorized signature file references | Restricted upload/read and association with signer |

Use D1 **foreign keys, unique indexes and check constraints**. Define indexes on owner, status, study date, revision and workflow review queue. Store canonical decimal **text** as entered/normalized for source fidelity, and parse under the versioned numeric algorithm. Validate finite numerical values and decimal formats at the boundary. Do not store observations only in an opaque JSON blob.

Readings, decisions, references and audit are normalized rows. JSON can hold **immutable derived result/policy snapshots**, with method version, input hash and a documented schema version.

## 6. Server-side modular boundaries

- **identity:** validate Access JWT, map active user, enforce Admin/Supervisor/Operator and ownership.
- **studies:** create/search/history, Draft changes, revisions, soft delete, multi-step data entry.
- **reference-custody:** Supervisor-only Attribute reference values and verification. No Operator reference fields in response payloads.
- **calculations:** independent Variable A&R and Attribute pooled-per-appraiser Kappa implementation.
- **policies:** approved policy snapshots and dispositions; numeric client risk criteria are not invented.
- **approvals:** review queue, Request Correction, supervisor-only approve/finalize with optimistic concurrency.
- **reports:** typed report snapshots, HTML print view, PDF rendering, private R2 archival and authorized download.
- **audit:** append-only business events for source edits, policy activation, approval, revision, report access where required.

Never reuse frontend-only role hiding as a security boundary; **check role + owner + revision state + blind-study publication on every API action**.

## 7. Calculation lifecycle and validity

1. Owner can save incomplete Draft; each meaningful change increments a revision input version and invalidates its previous result.
2. Calculate only after relevant data completeness: Variable **90/90**; Attribute **300/300 + 50 verified references**, distinct appraisers and valid setup.
3. Compute **on the server**, using pure versioned methods. Validate data again at the Worker boundary; frontend-only math is preview at most.
4. Save input hash, full-precision metrics, method version, policy version/snapshot and warnings. Never present NaN/Infinity; use explicit "Not Calculable" with reasons.
5. Variable: show corrected EV/AV/GRR/PV/TV, Ratama %Study Var, separate %Tolerance, NDC floor convention and R-chart checks. Attribute: show pooled Kappa, accuracy, within-appraiser agreement, both-trials-correct, missed-NG/false-reject with denominators.
6. A missing/invalid Quality policy yields **Review Required**, not automatic OK. No unapproved all-appraiser verdict.
7. Submit for Approval requires a **current** run matching the exact current input hash. Only the Supervisor may approve/finalize an authorized disposition.

Golden fixtures from the audited Ratama workbooks must be reproduced independently before accepting engine implementation, including negative AV² clamping, undefined TV/GRR, Kappa denominator Pe=1, class-absence risks and unrounded threshold checks.

## 8. Approval → immutable report: safe sequence

**D1 and R2 do not share a distributed transaction.** Use a controlled two-stage operation instead of "UPDATE finalized, then hope PDF succeeds":

1. Under Supervisor RBAC, validate the latest input hash, policy approval evidence, metadata and available authorized signature asset. Lock approval intent with an idempotency token / version predicate.
2. Render a **candidate PDF** from the exact method/policy/input snapshot and check the generated bytes/content hash.
3. Upload PDF to private R2 with a **new, unique revision-and-hash key**. Do not overwrite an existing final artifact. Check readback/metadata and apply appropriate retention only after its policy is approved.
4. Commit one **D1 atomic batch** for report metadata, approval, audit and Draft → Finalized. Require a **database-enforced compare-and-swap/version guard** (constraint/trigger or equivalent checked transaction path) that fails the transaction if the revision was changed or is already Finalized; checking a prior SELECT alone is insufficient. Verify only one approval/report link can exist per finalized revision. If a concurrent edit or DB commit fails, do not expose Finalized; reconcile/delete stale *staging* objects with safe cleanup. Handle R2-write success + D1-failure explicitly.
5. Release archived R2 PDF only through an authenticated, permission-checked Worker route; return the **stored bytes**, not a fresh render with a future template.
6. A subsequent change requires Supervisor **Create Revision**, not editing the old final source or overwriting the old PDF.

For R2 final artifacts, consider **Bucket Lock retention** when Ratama decides legal and operational retention duration; a configurable lock protects against ordinary overwrite/delete but **is not automatically a certified legal WORM or digital-signature system**. A scanned/drawn signature image is visual evidence only; approval legitimacy comes from the authenticated audit event. Never forge an absent logo or signature.

### 8.1 PDF implementation gate

Start with print-only HTML/CSS, A4 margins, logo at header, clear metric and warning tables, method/policy versions and an authorized Supervisor sign-off block. Browser Run's /pdf quick action is suitable for HTML-to-PDF but requires a staging proof of concept for fonts, page breaks, mobile-preview consistency, quotas, costs and secure handling before final choice.

## 9. Candidate HTTP API

| Endpoint | Permission |
| --- | --- |
| GET /api/me | Verified Access user |
| GET /api/studies | All authenticated Operator/Supervisor; apply blind-data redaction |
| POST /api/studies | Operator, Supervisor |
| GET /api/studies/:id | Owner/Supervisor full Draft; other Operators read-only restricted metadata; Final report access based on visibility policy |
| PATCH /api/studies/:id/revisions/:rev | Owner of current Draft or Supervisor, optimistic concurrency |
| PUT /api/studies/:id/revisions/:rev/readings | Owner Draft/Supervisor, validated batches |
| PUT /api/studies/:id/revisions/:rev/decisions | Owner Draft/Supervisor, validated batches |
| GET,PUT /api/studies/:id/revisions/:rev/references | Supervisor **only** |
| POST /api/studies/:id/revisions/:rev/calculate | Owner or Supervisor; blind-result restrictions |
| POST /api/studies/:id/revisions/:rev/submit | Owner Draft / Supervisor |
| POST /api/studies/:id/revisions/:rev/request-correction | Supervisor |
| POST /api/studies/:id/revisions/:rev/approve-finalize | Supervisor; Quality policy and current snapshot required |
| POST /api/studies/:id/revisions | Supervisor, based on finalized parent |
| GET /api/studies/:id/revisions/:rev/report | Authenticated permitted viewer, private R2 proxy |
| GET,POST /api/admin/users | Admin only |
| GET,POST /api/supervisor/policies | Supervisor, activation requires documented Quality approval |

Names and exact payload schemas are proposals, not final API contracts. Use cursor/pagination and filters for History; structured errors for missing cells, unauthorized reference access and stale results.

## 10. Source layout and release process

~~~text
repo/
  apps/
    web/                 # React SPA, desktop/mobile
    api/                 # Hono Worker and domain handlers
  packages/
    calculations/        # Pure verified calculations + golden tests
    contracts/           # Zod schemas, discriminated result types
    ui/                  # Shared approved visual components
  db/
    migrations/          # D1 schema + tested migrations
  docs/
    03-calculations/
    04-ui-ux/
    05-architecture/
    06-quality-assurance/
  tests/
    e2e/
    integration/
~~~

One monorepo and **one deployed Worker application**. The repository layout is for modular boundaries; it does **not** imply microservices. CI: lint/typecheck/unit golden fixtures → D1 migration test → security/role integration tests → Playwright key E2E → staging deploy → reviewed production deploy. Separate D1 databases and R2 buckets for staging versus production. Never publish real client measurement records in the public GitHub repo.

## 11. Operations and resilience

- Cloudflare Access OTP and application role enforcement; authenticated API requests verify JWT claims, never trust an email string sent by the browser. Apply origin/CSRF checks to state-changing browser API requests, consistent cookie/session handling, security headers and rate limits.
- No public R2 bucket; restrict document downloads by app study visibility and retain audit trail.
- D1 point-in-time Time Travel is **7 days on Workers Free / 30 days on Workers Paid** (as of October 2026), not a substitute for long-term independent backups; schedule reviewed encrypted exports to a controlled storage location if retention requires it.
- Daily/weekly backups, migration rollback, R2 asset recovery, retention, log redaction, user offboarding, least-privilege secrets, and staging-production isolation require actual operational runbooks.
- D1 read/write limits, Browser Run usage and R2 storage charges must be validated against anticipated study volume and chosen Cloudflare plan before launch.
- Prefer UTC for stored event timestamps and explicit Asia/Jakarta rendering for business display.
- One client does not require a tenant_id on every table initially; add multi-tenancy only through a separate reviewed migration if product scope changes.

## 11.1 Why D1 instead of PostgreSQL at MVP?

D1 is the **proposed default**, not a claim that SQLite is universally better. For modest numbers of users, controlled report generation, one client and a limited relational schema, D1 reduces operational services. Switch to managed PostgreSQL if validated requirements demand complex transactional workflows outside safe D1 patterns, large-scale concurrent write contention, enterprise integration/SQL extensions or multi-client expansion. Avoid introducing a second database merely for hypothetical scale.

## 12. Technical risks and gates before implementation

| Gate | Why it matters | Status |
| --- | --- | --- |
| Access email OTP replaces S01 password field | UI specification and identity model conflict must be reconciled | **OPEN** |
| Ratama Quality approval of Attribute risk thresholds and composite policy | Avoid fabricated NG-miss/false-reject acceptance criteria | **OPEN** |
| Detailed report and real logo/signature assets | No actual report signature can yet be issued | **OPEN** |
| PDF proof of concept + retention policy | Confirm print quality, page breaks, R2/DB consistency | **OPEN** |
| D1 schema/ERD and endpoint contract review | Verify completeness, concurrency and permission assumptions | **OPEN** |
| Golden calculation regression and reference blindness security | Critical mathematical and operational safety gates | **OPEN** |
| Final high-fidelity visual polish | v0.1 UX accepted in concept, palette not accepted | **OPEN** |

**Scope of this document:** Technology selection has been accepted in **DEC-033**, but the ERD, API contract, security implementation, PDF renderer, backups and calculation acceptance policies are still proposals requiring their own verification/review. The accepted stack does **not** authorize an untested production deployment. Preserve the detailed architecture as DRAFT until its review gates pass.
