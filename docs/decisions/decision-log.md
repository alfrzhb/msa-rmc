# Decision Log — RMC MSA Analysis Tool

**Document status:** DRAFT / LIVE REGISTER  
**Last reviewed:** 29 October 2026 (ERD + Dictionary v0.2 corrective design review)  
**Basis:** Requirements Analysis v0.3 and stakeholder feedback

This log distinguishes an item accepted in a requirements review from a full, formally `FROZEN` specification.

## Accepted in review (NOT FROZEN)

| ID | Decision | Rationale / impact | Approval scope |
| --- | --- | --- | --- |
| DEC-001 | Start with one client company, defer multi-client management | Keep MVP simple; design for future extensibility | Business direction |
| DEC-002 | Login with individual employee accounts | Traceability and operator accountability | Business direction |
| DEC-003 | Three user roles: Admin, Supervisor, Operator | Separate account administration from quality operations | Reviewed and agreed |
| DEC-004 | Operator can edit own study; Supervisor can manage studies across operators | Controlled editing and supervision | Reviewed and agreed |
| DEC-005 | Manual input and paste-from-Excel for MVP; full XLSX import deferred | Minimize complexity and speed up use | Reviewed and agreed |
| DEC-006 | Finalized studies are not edited in place; changes use a revision | Preserve historical results and traceability | Reviewed and agreed |
| DEC-007 | MSA Variable and Attribute first; SPC later | Focus on validated scope | Business direction |
| DEC-008 | Study history and printable/exportable PDF reports | Core client needs | Business direction |
| DEC-009 | Store evolving specifications as Markdown in GitHub | Reduce risk of losing requirements history | Documentation workflow |
| DEC-010 | Use Ratama workbooks and training as the functional/reference baseline, but recalculate using validated mathematical formulas | Avoid propagating workbook arithmetic defects or treating the old v1.0 freeze as approval for this rebuilt project | Reviewed and agreed direction |
| DEC-011 | Variable MVP uses Crossed Gage R&R Average & Range | Matches Ratama's Variable workbook; ANOVA and other methods deferred | Reviewed and agreed direction |
| DEC-012 | Use Ratama 10 parts × 3 appraisers × 3 trials as initial Variable preset; Attribute workbook 50 × 3 × 2 is a provisional preset | Keep initial UX simple; training describes other valid sizes, so these are not universal limits | Variable accepted direction; Attribute candidate |
| DEC-013 | Correct erroneous AV and R-chart UCL calculations rather than reproduce spreadsheet displayed errors | Verifiable mathematical correctness is required | Reviewed and agreed direction |
| DEC-014 | Show Variable %Study Var, %Tolerance when specifications exist, NDC and independent range-control information | Ratama training explicitly distinguishes %Study Var and %Tolerance | Reviewed and agreed direction |
| DEC-015 | Record audit findings, method assumptions, golden figures and remaining validation in Markdown; never publish raw client datasets | Traceability and data protection | Documentation workflow |
| DEC-016 | Attribute pooled Kappa vs Reference plus agreement/error indicators is accepted as a candidate for final review | Preserves Ratama workbook and flags missed NG; still requires domain criteria approval | Candidate method direction agreed, NOT FROZEN |
| DEC-017 | Ratama pooled per-appraiser Kappa differs from mean of per-trial known-standard Kappa as documented by Minitab | Do not mislabel analytical methods or claim identical software results | Independent methodological audit |
| DEC-018 | Separate Attribute per-decision accuracy, per-sample all-trials-correct, and within-appraiser repeatability | Prevent misleading agreement interpretation | Recommended for final review, NOT FROZEN |
| DEC-019 | Use Ratama pooled appraiser-vs-reference Kappa as MVP main Attribute metric, with distinct indicators and method/version label | User accepted recommendation G1; Minitab trial-mean Kappa remains a different method | Accepted for product design; Ratama domain approval open |
| DEC-020 | Variable status shows Ratama %GRR band, separate NDC/R-chart checks and %Tolerance; trigger Review Required on conflicts | Do not mask statistical warnings with one simple OK | Accepted for product design; customer decision policy open |
| DEC-021 | Adopt Ratama workbook threshold semantics and full-precision comparisons; literal floor for positive-GRR NDC raw | Keep source-compatible boundaries; distinguish Minitab low-end minimum-1 convention | Accepted for product design; domain review before FROZEN |
| DEC-022 | Use audited golden test vectors and edge-case plan for implementation; no test-run claim until code exists | Statistical reproducibility and error detection | Accepted testing approach |
| DEC-023 | Attribute MVP has one study with three appraiser tabs; one authorized owner/Supervisor enters all paper-collected data | Supports either combined paper or separate paper sheets without changing operator permissions | Recommended workflow; user reviewing |
| DEC-024 | Distinguish appraiser identity from logged-in data-entry identity; preserve per-cell audit attribution | Inspection performer and system editor are not necessarily the same person | Recommended workflow |
| DEC-025 | Apply Ratama/AIAG-aligned study collection guidance and applicable customer-specific MSA requirements | Standards-based design is not customer signoff | Accepted principle; client protocol validation open |
| DEC-026 | Attribute uses Standard Preset + Configurable Approved Policy: fixed validated formulas, Ratama-derived default judgment bands, versioned Supervisor-managed acceptance criteria requiring Quality approval | Keeps operator UX simple without inventing universal NG-miss/false-reject limits or altering historic results | User agreed to product design; domain thresholds NOT FROZEN |
| DEC-027 | Only Supervisor can Approve & Finalize studies; Operator may calculate, save and submit for approval | Single-level Supervisor sign-off, no manager hierarchy | Product-owner accepted for MVP design; NOT FROZEN |
| DEC-028 | Only Supervisor can create, edit and verify Attribute Known Reference | Prevent uncontrolled alteration and leakage during blind assessment | Product-owner accepted for MVP design; NOT FROZEN |
| DEC-029 | Operators can view all study History; only modify own Drafts, and finalized studies from others are read-only | Shared visibility with owner-based write access | Product-owner accepted for MVP design; NOT FROZEN |
| DEC-030 | Official Final PDF includes company logo and Supervisor approval/signature section | Approval record, identity, timestamp and authorized signature asset; branding assets to follow | Product-owner accepted for MVP design; asset/style/legal details OPEN |
| DEC-031 | Approve Responsive Wireframe v0.2 as the UI/UX design direction for desktop/tablet/mobile | Keep full desktop entry grids; mobile one-part/one-sample entry, compact navigation, card-based history; preserve role/approval flow | **Product-owner UX approval, 28 October 2026**; not statistical/Quality validation or entire project FROZEN |
| DEC-032 | High-Fidelity UI v0.1 accepted as a general layout/UX direction, **subject to visual refinement**: navy/teal color palette feels too generic/AI-styled | Preserve approved mobile/desktop patterns and usability; require restrained industrial styling, fewer decorative effects and accessible semantic colors before final visual approval | Product-owner qualitative feedback, 28 Oct 2026; **palette NOT approved**, entire spec NOT FROZEN |
| DEC-033 | Select MVP stack: React/Vite/TypeScript; Tailwind/shadcn; Hono + Cloudflare Workers Static Assets; **D1 (SQLite) + Drizzle**; Cloudflare Access OTP + backend RBAC; private R2; Zod/Vitest/Playwright; GitHub Actions/Wrangler. Preferred HTML/CSS + Browser Run PDF requires PoC | One-client modular monolith with managed infrastructure; **Hostinger and PostgreSQL not chosen for initial MVP**. Backend calculation and approval/audit remain versioned | **Product-owner accepted 28 Oct 2026 — tech stack direction only**. Detailed architecture, Quality criteria and production QA NOT FROZEN |

## Open decisions (DISCUSSION REQUIRED)

| ID | Question | Current proposal |
| --- | --- | --- |
| OPEN-001 | Attribute final method approval and naming | User accepts Ratama pooled Kappa vs Reference as design direction; Ratama/client quality-domain signoff still needed |
| OPEN-002 | Flexible vs locked part/appraiser/trial counts | Workbook configurations are initial presets; alternatives need verified constants, protocol and acceptance tests |
| OPEN-003 | Calculation version and final signoff | Source data outputs independently audited; test vectors in calculation documents; raw XLSX formula-cell scan and acceptance pending |
| OPEN-004 | Judgment, threshold boundary and %Tolerance policy | User accepts Ratama-based bands and Review Required for conflicts; customer-specific acceptance/composite policy signoff still required |
| OPEN-006 | Other Admin and Supervisor permission details | **Supervisor-only Approve & Finalize accepted**; known reference write/verify Supervisor-only; deletion/revision safeguards still need implementation review |
| OPEN-007 | Exceptional disposition and approval lifecycle details | Supervisor Approve & Finalize accepted; signed reports for approved studies, immutable prior revisions; treatment of NG, missing policy or critical warnings still needs Quality-approved rules |
| OPEN-008 | Detailed PDF design and approval assets | Logo and Supervisor signature/approval block accepted; actual logo/signature assets, signatory title, layout, appendix and any e-signature compliance remain pending |
| OPEN-009 | Infrastructure implementation details, backup, secure Access/JWT integration, Browser Run feasibility and deployment readiness | **Technology stack selected (DEC-033)**; these implementation checks remain OPEN, not a new hosting/database choice |
| OPEN-010 | Attribute numerical missed-NG / false-reject limits and whole-study verdict | Standard preset + versioned configurable APPROVED policy agreed as design; actual limits/customer approval and global verdict remain open |
| OPEN-011 | Low-end NDC convention domain check | Design uses Ratama literal floor(NDCraw) when GRR>0; no Minitab minimum-1 parity claim; Ratama reviewer confirmation pending |

## Product-owner approval/visibility decisions — 2026-10-23

- [Field & Screen Specification v0.3](../04-ui-ux/field-and-screen-specification-v0.3.md) records four accepted product decisions, **NOT FROZEN**.
- **DEC-027:** Supervisor alone can approve and finalize. Operator can calculate, save, and submit; approval recorded atomically with Finalize (one approval tier).
- **DEC-028:** Supervisor alone may manage/modify/verify the 50 Attribute Known Reference values; blind appraiser views are still shielded.
- **DEC-029:** Operators can view **all study history**, but may edit only their own Drafts. Other Draft source/reference data remain protected while blind studies are in progress; finalized reports are read-only for other Operators.
- **DEC-030:** A Final PDF must show approved company logo and Supervisor signature/approval provenance; do not fake missing signature assets or mislabel a pasted image as a certified digital signature.
- **Open:** Approval of risk thresholds/exception policy by Ratama Quality; real logo and signature assets; signature method/legal sufficiency; branded report layout; UAT and all-purpose specification freeze.

## Attribute Policy Agreement — 2026-10-23

- [Attribute Judgment Policy v0.1](../03-calculations/attribute-judgment-policy-v0.1.md): accepted **Standard Preset + Configurable Approved Policy** design, with formulas noneditable, Supervisor-only selection/configuration of Quality-approved criteria, policy version snapshot at Finalize, and no automatic OK for unapproved risk limits.
- Ratama's Kappa bands are the candidate preset; the final customer-approved missed-NG and false-reject numbers are not yet determined.

## Product Review — 2026-10-23

- [Calculation Decisions v0.3](../03-calculations/calculation-decision-review-v0.3.md) — user's accepted G1/G2/G4/G6 direction and standards-aligned G5 constraints (not final domain signoff).
- [Attribute Study Input Workflow v0.1](../04-ui-ux/attribute-study-input-flow-v0.1.md) — proposed one-study/three-appraiser sections to handle one consolidated or three separate paper forms, without multi-user study-edit rights in the initial MVP.
- **Still open:** Ratama's missed NG / false-reject acceptance criteria; combined Attribute study judgement; final operational sample protocol and customer approvals; user review of input flow.

## Final Calculation Review — 2026-10-22

- [Final Calculation Review v0.2](../03-calculations/final-calculation-review-v0.2.md): both methods, corrected results, method/denominator differences and G1–G6 approval gates.
- [QA regression addendum v0.2](../06-quality-assurance/calculation-review-test-matrix-v0.2.md): method-parity, all-trials-correct and boundary tests.

## Audit record — 2026-10-22

- [Ratama source reconciliation](../03-calculations/ratama-source-audit-v0.1.md): workbook vs PPT, numerical deviations and interpretation risks.
- [Variable candidate formula specification](../03-calculations/variable-msa-calculation-spec-v0.1.md): matched Ratama Average & Range input methodology with corrected calculations.
- [Attribute Kappa audit](../03-calculations/attribute-msa-audit-v0.1.md): corrected reference agreement and missed-NG rates; approval pending.
- [Regression plan](../06-quality-assurance/calculation-test-plan-v0.1.md): expected values and pending executable tests.

**Decision boundary:** Decisions labeled “agreed direction” do not freeze all formulas, acceptance criteria or method variants. The old Ratama calculation v1.0 FROZEN document is historical evidence only.

## Change control

1. Add new decisions or change proposals to this log.
2. Update the affected Markdown document with a version bump.
3. Mark a document `FROZEN` only after explicit stakeholder approval.
4. Preserve the previous version in Git history; never silently rewrite a frozen baseline.

No documents in this repository are currently FROZEN.

## Field & Screen Revision v0.2 — 2026-10-23

- [Field & Screen Specification v0.2](../04-ui-ux/field-and-screen-specification-v0.2.md): **revised draft, NOT FROZEN**. Kept [v0.1](../04-ui-ux/field-and-screen-specification-v0.1.md) for comparison and history.
- Revised four UX points: Core/Advanced Details, Operator Submit for Review with **Supervisor-only Finalize as conservative proposal**, Attribute Known Reference restricted to separate Supervisor subview, and primary judgement/warnings before expandable calculation formulas.
- Review conditions remain OPEN: Ratama approval of Finalize authority and exception handling, reference custodian, Operator visibility, policy risk thresholds, and PDF branding; this document is not implementation approval.
- QA: 25 screen acceptance cases are specified, **not executed**.

## Field and Screen Draft Log — 2026-10-23

- [Field and Screen Specification v0.1](../04-ui-ux/field-and-screen-specification-v0.1.md) created using the current Ratama workbook templates, requirements, calculation decisions, and historical screen specification.
- Includes 13 screen types, a shared field dictionary, three appraiser tabs per MSA, roles, status transitions, policy selection, PDF contents and screen acceptance checks.
- This is a DRAFT review document, not a newly approved or frozen baseline.
- Pending: Attribute reference-entry authorization; operator Finalize rule; operator history visibility; PDF branding, and Ratama Quality acceptance thresholds.

## Wireframe produced — 2026-10-23

- [Wireframe Review v0.1](../04-ui-ux/wireframe-review-v0.1.md) records the 13-screen low-fidelity prototype, interaction matrix, browser/screenshot review and UX checkpoints. The interactive standalone HTML and screenshots were delivered in conversation, **not pushed as application source to GitHub**.
- This is a design artifact only; no statistical engine, real login, backend approval or production PDF exists.
- Approval/visibility decisions remain those in DEC-027 to DEC-030. The wireframe does not change or FROZEN-approve any calculations or customer Quality limits.

## Responsive Wireframe v0.2 review — 2026-10-28

- [Responsive Wireframe Review v0.2](../04-ui-ux/wireframe-responsive-review-v0.2.md) documents responsive desktop/tablet/mobile UI arrangements, focused per-part/per-sample mobile entry, bottom navigation, mobile History cards, and role-based approval/reference simulation.
- Browser QA of the standalone prototype covered 8 widths × 3 roles × 15 page states with **zero document horizontal overflows and zero JavaScript errors**. Simulated interactions were checked separately.
- HTML, screenshot and ZIP deliverables were provided in the conversation, **not committed as production source**. Do not interpret this UI review as customer signoff, frozen MSA acceptance policy or application implementation.

## Responsive UI/UX sign-off — 2026-10-28

- Product owner expressly **approved Responsive Wireframe v0.2** as the **UI/UX design baseline for the next visual design/implementation planning step** (DEC-031).
- Accepted pattern: desktop full input tables; mobile focused part/sample entry; responsive dashboard, History cards, results, and Supervisor approval flow.
- [Responsive Wireframe Review v0.2](../04-ui-ux/wireframe-responsive-review-v0.2.md) now records this scope-limited sign-off.
- Explicitly **not** approved by this UX decision: calculation-engine formulas, customer Quality judgement limits/exception policy, actual PDF signature assets, production security, test/UAT results or entire requirements freeze.

## High-fidelity visual design proposed — 2026-10-28

- [High-Fidelity UI Design Review v0.1](../04-ui-ux/high-fidelity-design-review-v0.1.md) proposes **Industrial Workspace** visual language, Teal/Navy palette, more legible mobile results and responsive app components atop the **already-approved** Responsive Wireframe v0.2.
- Standalone interactive prototype/screenshots delivered in conversation; **360 viewport × role × screen render cases passed with no global overflow or JavaScript page errors**.
- **Proposal only:** Product owner has not yet accepted high-fidelity visual details. Do **not** add an accepted DEC number or mark FROZEN at this stage. Domain calculations and Quality/customer policy remain separate pending validation.


## High-fidelity visual feedback — 2026-10-28 (DEC-032)

- The product owner said High-Fidelity UI v0.1 is broadly acceptable, but **the navy/teal coloring feels like a generic AI-generated dashboard**.
- **Accepted provisionally:** layout, responsive UX flow, data-centric industrial workspace direction.
- **Not accepted/finalized:** current color palette and full visual identity. Next visual pass should replace navy/teal-heavy surfaces with a muted, restrained enterprise/industrial direction; avoid gradients, excess shadows and decoration; use accessible semantic status colors.
- **Candidate only, not signed off:** off-white `#F8F9FA`, charcoal `#24292F`, muted steel blue `#456A81`; Inter or IBM Plex Sans. Ratama brand/logo assets have not been supplied.
- [High-Fidelity UI Design Review v0.1](../04-ui-ux/high-fidelity-design-review-v0.1.md) now includes the review and visual polish acceptance gate. Do not claim the prototype itself has been recolored.
- **Scope boundary:** No new acceptance of statistical formulas, Quality thresholds, role-policy implementation, production code or FROZEN document state.


## System Architecture candidate — 2026-10-28 (PROPOSED, NOT DECIDED)

- [System Architecture Design v0.1](../05-architecture/system-architecture-v0.1.md) recommends **React/Vite/TypeScript + Hono on one Cloudflare Worker + D1 + private R2 + Cloudflare Access OTP + Browser Run PDF**, as a **modular monolith** suitable for a single Ratama client.
- Proposal covers separate pure calculation module, per-study revision/source table model, server-enforced RBAC, Supervisor-only Known Reference/approval, immutable approved reports, and preview/staging/production isolation.
- **Technical review required:** S01 screen currently implies local email/password but proposed Access uses hosted OTP. App Admin roles and Access identity allowlisting require coordinated onboarding.
- D1/R2 finalize is not a distributed transaction: planned safe staged PDF + version-guarded D1 commit must be proven in implementation. Customer Quality criteria and electronic-signature/retention obligations remain open.
- **No approved architecture decision ID assigned yet**: user requested a recommended stack, not approval to freeze or start coding. No application code or cloud resources created.


## Technology stack selected — 2026-10-28 (DEC-033)

- Product owner explicitly selected **Cloudflare Workers + D1 (SQLite) + private R2** for the Ratama MSA MVP after considering Hostinger shared/VPS and external PostgreSQL. React/Vite/TypeScript, Tailwind CSS/shadcn/ui, Hono, Drizzle ORM, Cloudflare Access email OTP, server RBAC, Zod, Vitest, Playwright and GitHub Actions/Wrangler form the accepted baseline.
- [Tech Stack Decision v1.0](../05-architecture/tech-stack-decision-v1.0.md) documents selection, rejected initial alternatives, architecture boundaries and test gates. [System Architecture v0.1](../05-architecture/system-architecture-v0.1.md) remains the detailed **review draft**.
- **Cloudflare Browser Run** is the preferred PDF technology but requires staging validation; do not claim rendered PDFs, Access login, security enforcement or migrations have been implemented.
- **Not approved/frozen:** database ERD, API schema, customer Quality/risk criteria, calculation implementation, S01 login screen reconciliation, actual logo/signature assets, Visual Polish v0.2, cloud costs/backup and production deployment.


## Database ERD v0.1 prepared — 2026-10-29 (DRAFT / NOT FROZEN)

- [Database ERD v0.1](../05-architecture/erd-database-v0.1.md) and [Data Dictionary v0.1](../05-architecture/data-dictionary-v0.1.md) document a **15-table candidate relational model** for Cloudflare D1, with a [separate Mermaid source](../05-architecture/erd-v0.1.mmd).
- Consistent with DEC-027–030 (Supervisor-only finalize/reference; Operator all-history read-only for others; signed final PDF) and DEC-033 (Cloudflare D1/Workers/R2).
- Variable keeps **90** measurements; Attribute keeps **300** decisions and **50 separate** Supervisor-verified references. Distinguish appraisers from authenticated system editors.
- Study revisions and stored final PDF/artifacts are designed to be immutable. Partial Drafts, formula/policy versioning, source change invalidation, composite FKs, uniqueness of readings and approval requests, and approval/R2 failure recovery are documented.
- **18 proposed review scenarios**, including concurrency and cross-revision corruption, are specified but **not executed**.
- **No new product-owner acceptance, schema FROZEN status, migrations, database provisioning or production implementation claimed.** Next: stakeholder ERD review → D1 migration design → API Contract v0.1 and security/calc QA.


## ERD v0.2 corrective design — 2026-10-29 (DRAFT; no schema acceptance)

- The user asked to **fix all previously identified v0.1 ERD deficiencies**. Prepared [ERD v0.2](../05-architecture/erd-database-v0.2.md), [Data Dictionary v0.2](../05-architecture/data-dictionary-v0.2.md), [Mermaid v0.2](../05-architecture/erd-v0.2.mmd). Kept v0.1 files unchanged for historical comparison.
- Retains 15 entities; adds selected_policy_id, current_input_sha256, audit old/new fields, Attribute OPEN/LOCKED/DISCLOSED, SUPERSEDED Draft state, source version invalidation, current policy checks, signer-owner composite FK, strict run/review/approval relationships, report idempotency and D1/R2 failure handling.
- Documents **20 proposed test scenarios**, including row-level composite FK errors, wrong signer, false reference verification, blind leakage, duplicate finalize and R2 orphan recovery.
- **Status:** Architectural **DRAFT**, **not** tested D1 migrations or validated statistical/customer policy, and not a new DEC or project-wide FROZEN milestone. Next gate is ERD design approval, then implement and test D1 constraints/triggers and API Contract.
