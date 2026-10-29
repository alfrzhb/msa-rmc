# RMC MSA — Database ERD v0.2 (Corrected)

**Status:** DRAFT / TECHNICAL REVIEW — NOT FROZEN  
**Date:** 29 October 2026  
**Chosen stack:** Cloudflare D1 (SQLite), Drizzle ORM, Cloudflare Workers + Hono (DEC-033)  
**Companion:** [Data Dictionary v0.2](data-dictionary-v0.2.md) · [Mermaid source](erd-v0.2.mmd) · [v0.1 archive](erd-database-v0.1.md)

## 1. Summary and scope

**All nine findings from ERD v0.1 are addressed in the v0.2 design** without expanding beyond **15 tables**. This is an updated logical/relational design, not a deployed D1 schema and not yet a passing migration or trigger suite.

| # | Previous gap | Change in v0.2 | Enforcement plan |
| --- | --- | --- | --- |
| 1 | Study had no selected policy | study_revisions.selected_policy_id + current_input_sha256 | FK, policy applicability/approval check and invalidation |
| 2 | Valid FKs could still cross revisions | Composite FKs joining revision, run and review request | UNIQUE targets + composite FK |
| 3 | Concurrent/double Finalize unsafe | One atomic guarded approval/report/final state batch, idempotency key | BEFORE/AFTER triggers + UQ + failure tests |
| 4 | Attribute blind process could leak results | OPEN → LOCKED → DISCLOSED; no source edit after exposure | Role + state checks + audit + source immutability |
| 5 | Finalized state not protected | FINALIZED/SUPERSEDED source immutable | Backend, reviewed update/delete triggers |
| 6 | Cell audit did not retain before/after | audit_events field, old value, new value, sensitivity | One transaction with source edit/version bump |
| 7 | Reference label could retain false verification | Label edit clears verified_by/verified_at | Trigger + Supervisor-only verification |
| 8 | Signature asset may be wrong owner/type | Composite signer/asset FK plus LOGO/SIGNATURE CHECK | FK, CHECK, report logo trigger |
| 9 | Study size editable without validated method | Explicit supported preset and calculation version gate | Zod + domain checks + QA |

A **new revision status SUPERSEDED** is only for a disclosed but not finalized Draft that can no longer be treated as blind. This does **not** authorize silently modifying an existing official result.

## 2. Corrected ER diagram

The diagram below is renderable in GitHub Markdown. Additional composite FK constraints are described immediately below rather than drawing every parallel relationship.

~~~mermaid
erDiagram
    USERS ||--o{ STUDIES : owns
    STUDIES ||--|{ STUDY_REVISIONS : versions
    STUDY_REVISIONS o|--o{ STUDY_REVISIONS : parent_of
    JUDGMENT_POLICIES o|--o{ STUDY_REVISIONS : selected_by
    STUDY_REVISIONS ||--o{ APPRAISERS : assigns
    STUDY_REVISIONS ||--o{ SAMPLES : defines
    APPRAISERS ||--o{ VARIABLE_READINGS : takes
    SAMPLES ||--o{ VARIABLE_READINGS : measured
    APPRAISERS ||--o{ ATTRIBUTE_DECISIONS : assesses
    SAMPLES ||--o{ ATTRIBUTE_DECISIONS : classified
    SAMPLES ||--o| ATTRIBUTE_REFERENCES : known_reference
    USERS o|--o{ ATTRIBUTE_REFERENCES : verifies
    STUDY_REVISIONS ||--o{ CALCULATION_RUNS : computes
    JUDGMENT_POLICIES o|--o{ CALCULATION_RUNS : applies
    STUDY_REVISIONS ||--o{ REVIEW_REQUESTS : submits
    CALCULATION_RUNS ||--o{ REVIEW_REQUESTS : submitted_result
    REVIEW_REQUESTS ||--o| APPROVALS : reviewed_into
    CALCULATION_RUNS ||--o| APPROVALS : approved_result
    STUDY_REVISIONS ||--o| APPROVALS : finalizes
    USERS ||--o{ APPROVALS : signs_off
    BRAND_ASSETS o|--o{ APPROVALS : signer_image
    APPROVALS ||--o| REPORT_ARTIFACTS : issues
    BRAND_ASSETS o|--o{ REPORT_ARTIFACTS : logo
    USERS o|--o{ AUDIT_EVENTS : actor
    STUDIES o|--o{ AUDIT_EVENTS : history
    STUDY_REVISIONS o|--o{ AUDIT_EVENTS : changed_revision

    USERS {
        text id PK
        text access_sub UK
        text email_norm UK
        text role
        int is_active
    }
    STUDIES {
        text id PK
        text study_code UK
        text msa_type
        text owner_user_id FK
        text deleted_at
    }
    STUDY_REVISIONS {
        text id PK
        text study_id FK
        int revision_no
        text parent_revision_id FK
        text selected_policy_id FK
        text lifecycle_status
        int input_version
        text current_input_sha256
        text attribute_lock_status
        text blind_closed_at
        text reference_disclosed_at
    }
    APPRAISERS {
        text id PK
        text revision_id FK
        int slot_no
        text normalized_name
    }
    SAMPLES {
        text id PK
        text revision_id FK
        int slot_no
        text normalized_label
    }
    VARIABLE_READINGS {
        text id PK
        text revision_id FK
        text appraiser_id FK
        text sample_id FK
        int trial_no
        text value_decimal
    }
    ATTRIBUTE_DECISIONS {
        text id PK
        text revision_id FK
        text appraiser_id FK
        text sample_id FK
        int trial_no
        text decision
    }
    ATTRIBUTE_REFERENCES {
        text id PK
        text revision_id FK
        text sample_id FK
        text known_label
        text verified_by_user_id FK
        text verified_at
    }
    JUDGMENT_POLICIES {
        text id PK
        text policy_key
        int version_no
        text status
        text applicability
        text rules_json
    }
    CALCULATION_RUNS {
        text id PK
        text revision_id FK
        text policy_id FK
        int input_version
        text input_sha256
        text method_version
        text results_json
    }
    REVIEW_REQUESTS {
        text id PK
        text revision_id FK
        text calculation_run_id FK
        int request_no
        text status
    }
    APPROVALS {
        text id PK
        text idempotency_key UK
        text revision_id FK
        text review_request_id FK
        text calculation_run_id FK
        text approved_by_user_id FK
        text signature_asset_id FK
        text approved_at
    }
    REPORT_ARTIFACTS {
        text id PK
        text approval_id FK
        text logo_asset_id FK
        text r2_key UK
        text sha256_hex
    }
    BRAND_ASSETS {
        text id PK
        text asset_kind
        text r2_key UK
        text signer_user_id FK
        text content_sha256
    }
    AUDIT_EVENTS {
        text id PK
        text study_id FK
        text revision_id FK
        text actor_user_id FK
        text actor_kind
        text sensitivity
        text before_json
        text after_json
        text reason
    }
~~~

**Key relationships:** Study → many revisions. Each revision → own Appraisers, Samples, source cells and calculation runs. One revision may have **zero or one** final Supervisor approval and an approval may have **zero or one** report until the atomic Finalize transaction; Finalized requires **exactly one of each**. Attribute reference and response rows never share a mutable source table.

## 3. Input cardinality, sparse Drafts and revision rules

| Study type | Appraisers | Samples | Trials/appraiser/sample | Full source rows | Other |
| --- | ---: | ---: | ---: | ---: | --- |
| Variable Average & Range | 3 | 10 | 3 | **90** | No Attribute Known Reference |
| Attribute agreement / pooled Kappa | 3 | 50 | 2 | **300** | **50 separately verified Known References** |

While DRAFT, record only filled source cells. Calculate requires **exact slot coverage**, not merely row counts. Each source row is unique by (revision_id,appraiser_id,sample_id,trial_no); references are unique by (revision_id,sample_id). The identity of the appraiser who inspected must remain distinct from the logged-in employee who keyed in a paper form.

**Lifecycle:**

~~~text
STUDIES (permanent study identity)
  └── STUDY_REVISIONS v1 [DRAFT]
        ├── Variable: 90 source readings at completion
        └── Attribute: 300 decisions + 50 supervisor-verified labels
              OPEN --[Supervisor closes all answers]--> LOCKED
              LOCKED --[authorized reference evaluation]--> DISCLOSED
        |
        ├── Calculate → append-only CALCULATION_RUNS
        ├── Submit → REVIEW_REQUESTS [PENDING]
        ├── Request correction → invalidate run; resubmit when corrected
        ├── Approve & Finalize → APPROVALS + REPORT_ARTIFACTS + FINALIZED
        └── After premature disclosure needing re-collection:
            Supervisor SUPERSEDE + reason → create v2 as new DRAFT
~~~

**Only one active DRAFT** per study. After an existing revision is FINALIZED, a new revision is created; the old source rows, approval and PDF remain immutable. If a blind Attribute Draft becomes DISCLOSED, **do not reopen or overwrite answers**; issue a new independent revision if corrections affect the already-disclosed classification process.

## 4. Relational integrity contract

| Relationship | Composite FK / uniqueness | Failure prevented |
| --- | --- | --- |
| Revision ancestry | UQ study_revisions(study_id,id); FK (study_id,parent_revision_id) → same table | Parent from another study |
| Reading → appraiser | UQ appraisers(revision_id,id); FK readings(revision_id,appraiser_id) | Appraiser from another revision |
| Reading → sample | UQ samples(revision_id,id); FK readings(revision_id,sample_id) | Sample from another revision |
| Decision → appraiser/sample | Same composite FKs as readings | Cross-revision classification |
| Reference → sample | FK references(revision_id,sample_id) → samples(revision_id,id) | Reference attached to another revision |
| Review → calculation | UQ runs(revision_id,id); FK review(revision_id,calculation_run_id) | Submitted run from wrong revision |
| Approval → exact submitted run | UQ reviews(revision_id,id,calculation_run_id); FK approval(revision_id,review_request_id,calculation_run_id) | Approval mismatched with request/run |
| Approval → calculation | FK approval(revision_id,calculation_run_id) → runs(revision_id,id) | Wrong revision result |
| Approval → authorized signer | UQ brand_assets(id,signer_user_id); FK approval(signature_asset_id,approved_by_user_id) | Other user's signature image |
| Final report → approval | UQ approvals(revision_id); UQ reports(approval_id); UQ reports(r2_key) | Duplicate official PDFs/approvals |

**SQLite CHECK cannot do cross-table role or applicability lookups.** Use reviewed triggers and server checks for matching MSA type, actual SUPERVISOR, policy state, blind state and report branding. Adding a FK alone does not enforce that a referenced policy is APPROVED.

Representative migration shapes, **illustrative only**:

~~~sql
CREATE UNIQUE INDEX uq_one_active_draft
  ON study_revisions(study_id) WHERE lifecycle_status = 'DRAFT';

CREATE UNIQUE INDEX uq_one_pending_review
  ON review_requests(revision_id) WHERE status = 'PENDING';

CREATE UNIQUE INDEX uq_review_run_tuple
  ON review_requests(revision_id, id, calculation_run_id);

-- In approvals CREATE TABLE:
-- FOREIGN KEY (revision_id, review_request_id, calculation_run_id)
-- REFERENCES review_requests(revision_id, id, calculation_run_id);
-- FOREIGN KEY (revision_id, calculation_run_id)
-- REFERENCES calculation_runs(revision_id, id);
-- FOREIGN KEY (signature_asset_id, approved_by_user_id)
-- REFERENCES brand_assets(id, signer_user_id);

-- In brand_assets CREATE TABLE:
-- CHECK (
--  (asset_kind='LOGO' AND signer_user_id IS NULL)
--   OR
--  (asset_kind='SIGNATURE' AND signer_user_id IS NOT NULL)
-- );
~~~

## 5. Calculation freshness and policy selection

- study_revisions.selected_policy_id identifies the exact selected policy version; this can be NULL in incomplete Draft. A selected policy must match study type, with documented Quality approval before it authorizes Finalize.
- **Every input-affecting edit** (measurement, answer, sample/appraiser mapping, reference verification, selected policy, applicable study setup) atomically increments input_version and clears current_input_sha256. It also invalidates any pending review, with audit evidence.
- Calculate performs consistent-read normalization and hashes an ordered input snapshot; only a version-checked write may set revision.current_input_sha256 and append a matching calculation_run containing input_version, input_sha256, method_version, policy snapshot, results and warnings.
- A run is CURRENT only when both revision.input_version and revision.current_input_sha256 match the saved run. Recalculate after edits; rounding display must not affect threshold comparisons.
- Without a signed Quality policy for acceptance thresholds, preserve independently calculated metrics and show **REVIEW_REQUIRED**, not fabricated PASS. Finalization is blocked unless Quality has approved an explicit applicable exception/disposition process.
- Attribute cannot expose reference-dependent Kappa, confusion metrics, missed NG, false reject or per-sample correctness to the Operator while the decisions are OPEN. Supervisor verification of references is separate from input lock.

## 6. Protected blind-study transitions

| Transition | Who | Guard | Audit and effect |
| --- | --- | --- | --- |
| OPEN → LOCKED | Supervisor | 300/300 answers complete, distinct slots and valid setup | blind_closed_at, actor, input version, BLIND_LOCKED |
| LOCKED → OPEN (before any disclosure) | Supervisor | Documented source/transcription correction and no reference-derived result exposed | reason, version increment, invalidated calculations |
| LOCKED → DISCLOSED | Supervisor | 50/50 verified known labels and current independently entered inputs | reference_disclosed_at + actor, DISCLOSED event |
| DISCLOSED → OPEN | Nobody | **Prohibited** | Supervisor must SUPERSEDE old Draft and start a new independent revision |
| FINALIZED/SUPERSEDED → editable | Nobody | **Prohibited** | Revision creation only |

Reference label edits clear verification and require re-verification, never simply preserve the old verified_by/time when the known label changes. Active references and **restricted audit before/after values** are never readable by Operator, including all-study History and other people's Drafts.

## 7. Atomic Supervisor Finalize with D1 + R2

D1 transactions cannot include R2 objects. **Render and validate PDF before committing Finalized**, then upload to a unique private R2 key and verify bytes/metadata/hash. A never-committed object remains hidden from the app and is reconciled as an orphan.

The database portion is a **single D1 batch transaction**:

1. Insert APPROVALS with idempotency_key and **BEFORE INSERT guard trigger**: require valid authenticated Supervisor actor, revision DRAFT and input_version/current_input_sha256 matching submitted CALCULATION_RUNS, exact PENDING review and same-run composite FKs, selected APPROVED type-compatible policy and authorized signatory asset. A mismatch triggers an actual SQL error; never rely on UPDATE matching zero rows.
2. Insert REPORT_ARTIFACTS referencing the just-created approval and the pre-verified unique R2 PDF. A guarded **AFTER INSERT report trigger** can perform the revision Finalize transition and assert precisely one row changed. An **AFTER UPDATE revision trigger** must require corresponding approval and report to exist. Trigger ordering/feasibility must be validated in real D1 migrations.
3. Set REVIEW_REQUESTS.status = APPROVED and append an AUDIT_EVENTS FINALIZED event, using guarded SQL. The whole batch must fail on any invalid transition, so no partial final state becomes visible.
4. On retry, use the idempotency key + UNIQUE constraints to return the already recorded report when present (rather than issuing duplicate signatures). If upload succeeded but D1 failed, reconcile the orphan R2 object and return a non-finalized error.

**Critical detail:** D1.batch guarantees transactional rollback when a statement **errors**; a filtered UPDATE affecting zero rows is not automatically an error. A reviewed trigger using SQLite RAISE(ABORT) or equivalent invariant check must enforce state and version transitions. See the [Cloudflare D1 batch API](https://developers.cloudflare.com/d1/worker-api/d1-database/) and [D1 foreign-key reference](https://developers.cloudflare.com/d1/sql-api/foreign-keys/).

**This is not an executed trigger implementation**. The D1 migration/proof-of-concept must test mismatched source digest, race between submit and edit, duplicate Finalize, wrong signer, wrong policy, absent PDF object, failed R2 upload, and D1 rollback.

## 8. Audit, immutability and read permissions

- audit_events now has actor_kind/actor_user_id, entity_type/id, field_path, before_json/after_json, sensitivity and reason. Exact source correction history is captured; it is not a generic log that loses prior values.
- Reference edits emit RESTRICTED_REFERENCE events, never displayed via general History to Operators. Staff/account/policy events can have null study_id and revision_id.
- Protect UPDATE/DELETE of FINALIZED and SUPERSEDED source rows, approvals, report_artifacts, relevant immutable policy content and audit events with service guards and reviewed SQL triggers. Source mutation + revision version bump + audit append belong to one atomic batch.
- Supplied signature image can be used only for the recorded authenticated Supervisor, using brand_assets type/owner guard; this is visual approval, **not a certified digital signature**. Report logo must be a LOGO asset; referenced assets must not be overwritten after issuance.
- All report downloads proxy private R2 through a role-checked API. Store immutable report byte SHA-256, method version, policy snapshot, signatory name/title, document template version and UTC approval time.

## 9. Revised QA gates (not executed)

| ID | Test scenario | Expected |
| --- | --- | --- |
| ER2-01 | Create partial Variable Draft with 5/90 measurements | Valid; empty slots absent |
| ER2-02 | Attempt source row with appraiser/sample from another revision | Composite FK rejects |
| ER2-03 | Attempt parent_revision_id belonging to another study | Composite self-FK rejects |
| ER2-04 | Calculate Variable with 89/90, then 90/90 | Incomplete blocked, complete eligible |
| ER2-05 | Create Attribute Draft with 300 answers, 49 verified references | No reference-derived result until 50 verified |
| ER2-06 | Operator requests Known Reference or RESTRICTED_REFERENCE audit | Denied without leakage |
| ER2-07 | Change known_label but retain verified_by/at | Trigger rejects; reverification mandatory |
| ER2-08 | Request Attribute Kappa while OPEN | Denied until authorized disclosure after lock |
| ER2-09 | Edit Attribute answer after DISCLOSED | Denied; must SUPERSEDE and use new independent revision |
| ER2-10 | Calculate, change selected_policy_id and submit old run | Stale version/hash rejected |
| ER2-11 | Approve wrong revision / wrong review request's run | Composite FK rejects |
| ER2-12 | Approve with another Supervisor's signature image | Composite FK/CHECK rejects |
| ER2-13 | Report references SIGNATURE instead of LOGO | Trigger/service rejects |
| ER2-14 | Parallel duplicate Finalize requests with same idempotency | Exactly one approval + PDF |
| ER2-15 | Edit between Calculate and Finalize | Guard raises error and entire batch rolls back |
| ER2-16 | R2 upload succeeds but D1 approval fails | Not Finalized; orphan reconciled |
| ER2-17 | Try changing finalized reading, policy snapshot, PDF metadata or audit event | Rejected |
| ER2-18 | Global Admin role change audit with no study_id | Valid and access-restricted |
| ER2-19 | Missing customer-approved missed-NG/false-reject criteria | REVIEW_REQUIRED, no fabricated PASS |
| ER2-20 | Supervisor correction before LOCKED result disclosure | Unlock with reason, run invalidated and re-lock after corrected answers |

**Important:** These are acceptance tests to implement in migration/API work, not tests that have passed.

## 10. Review disposition and next step

**Design fixes implemented in documentation:** all nine audit items. **Implementation pending:** Drizzle migrations for all 15 tables, D1 composite FK and trigger tests, exact JSON contracts, Access JWT/RBAC, input-hash concurrency, R2/PDF finalization, Quality-approved thresholds and UI S01 OTP reconciliation.

**Next proposed milestone:** Review ERD v0.2 → approved schema review gate → D1 migration and executable constraint tests → API Contract v0.1. No project-wide FROZEN claim.
