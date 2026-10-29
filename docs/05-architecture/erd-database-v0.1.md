# RMC MSA — Database ERD v0.1

**Status:** ARCHITECTURE REVIEW DRAFT — NOT FROZEN  
**Date:** 29 October 2026  
**Accepted stack:** Cloudflare D1 (SQLite) + Drizzle ORM; TypeScript / Hono on Workers (DEC-033)  
**Companion files:** [Full Data Dictionary v0.1](data-dictionary-v0.1.md) · [Raw Mermaid diagram](erd-v0.1.mmd) · [System Architecture v0.1](system-architecture-v0.1.md)

## 1. Objective and design boundary

This is the **first implementable candidate database model** for Ratama's single-client MSA MVP. It supports partial Drafts, exact Variable/Attribute source inputs, role-controlled Known Reference, deterministic calculation runs, versioned Quality policy, one-level Supervisor approval, immutable signed-report artifacts, and all-study History.

It is **not** a deployed database, an executed Drizzle migration, or a Quality-approved calculation release. The 15-table model is a deliberate small relational core rather than a general quality-management system. No tenant layer, SPC, import job queue or unnecessary generic field-value EAV table is added.

**Table groups:**

| Domain | Tables | Count |
| --- | --- | --- |
| Identity | users | 1 |
| Study / revision / input | studies, study_revisions, appraisers, samples, variable_readings, attribute_decisions, attribute_references | 7 |
| Calculation / policy | calculation_runs, judgment_policies | 2 |
| Approval / reporting | review_requests, approvals, report_artifacts, brand_assets | 4 |
| Audit | audit_events | 1 |
| **Total** | | **15** |

## 2. ER diagram

GitHub renders the Mermaid ERD directly in this document; editable diagram source is stored separately at [erd-v0.1.mmd](erd-v0.1.mmd).

~~~mermaid
erDiagram
    USERS ||--o{ STUDIES : owns
    STUDIES ||--|{ STUDY_REVISIONS : versions
    STUDY_REVISIONS ||--o{ APPRAISERS : assigns
    STUDY_REVISIONS ||--o{ SAMPLES : defines
    APPRAISERS ||--o{ VARIABLE_READINGS : measures
    SAMPLES ||--o{ VARIABLE_READINGS : measured_part
    APPRAISERS ||--o{ ATTRIBUTE_DECISIONS : assesses
    SAMPLES ||--o{ ATTRIBUTE_DECISIONS : assessed_sample
    SAMPLES ||--o| ATTRIBUTE_REFERENCES : has_known_reference
    USERS ||--o{ ATTRIBUTE_REFERENCES : verifies
    STUDY_REVISIONS ||--o{ CALCULATION_RUNS : calculates
    JUDGMENT_POLICIES o|--o{ CALCULATION_RUNS : evaluates_with
    STUDY_REVISIONS ||--o{ REVIEW_REQUESTS : submits
    CALCULATION_RUNS ||--o{ REVIEW_REQUESTS : submitted_result
    REVIEW_REQUESTS ||--o| APPROVALS : leads_to
    STUDY_REVISIONS ||--o| APPROVALS : finalized_by
    USERS ||--o{ APPROVALS : supervisor
    APPROVALS ||--|| REPORT_ARTIFACTS : issues
    BRAND_ASSETS o|--o{ APPROVALS : signs
    STUDIES o|--o{ AUDIT_EVENTS : recorded_for
    USERS ||--o{ AUDIT_EVENTS : acts
    STUDY_REVISIONS o|--o{ AUDIT_EVENTS : context

    USERS {
        text id PK
        text access_sub UK
        text email_norm UK
        text role
        boolean is_active
    }
    STUDIES {
        text id PK
        text study_code UK
        text msa_type
        text owner_user_id FK
        datetime deleted_at
    }
    STUDY_REVISIONS {
        text id PK
        text study_id FK
        int revision_no
        text parent_revision_id FK
        text lifecycle_status
        int input_version
        text study_name
        text characteristic
        datetime blind_closed_at
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
        text sample_label
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
        datetime verified_at
    }
    JUDGMENT_POLICIES {
        text id PK
        text policy_key
        int policy_version
        text status
        json rules_json
        text approval_evidence
    }
    CALCULATION_RUNS {
        text id PK
        text revision_id FK
        text policy_id FK
        int input_version
        text input_hash
        text method_version
        json results_json
    }
    REVIEW_REQUESTS {
        text id PK
        text revision_id FK
        text calculation_run_id FK
        text submitted_by_user_id FK
        text status
        datetime submitted_at
    }
    APPROVALS {
        text id PK
        text revision_id FK
        text review_request_id FK
        text calculation_run_id FK
        text approved_by_user_id FK
        text signature_asset_id FK
        datetime approved_at
    }
    REPORT_ARTIFACTS {
        text id PK
        text approval_id FK
        text r2_key UK
        text sha256_hex
        text content_type
    }
    BRAND_ASSETS {
        text id PK
        text asset_kind
        text r2_key UK
        text signer_user_id FK
        text sha256_hex
    }
    AUDIT_EVENTS {
        text id PK
        text study_id FK
        text revision_id FK
        text actor_user_id FK
        text action_type
        datetime happened_at
    }
~~~

**How to read:** USERS are *system accounts*; APPRAISERS are the actual humans who measure or inspect and **do not need their own application login**. A STUDY retains a stable ID and has versioned STUDY_REVISIONS; each revision has its own APPRAISERS, SAMPLES, input rows, calculation runs and independent review history. APPROVALS exist only when Supervisor approval is valid. R2 holds PDF bytes; REPORT_ARTIFACTS only stores their verified metadata and key.

Not every foreign key is drawn as a Mermaid relationship because 20+ links would reduce legibility; the [Data Dictionary](data-dictionary-v0.1.md) lists full FKs and ownership checks. An optional relationship in the drawing does not replace stricter **business-level finalization gates**.

## 3. Cardinality and study workflows

### 3.1 Variable study

~~~text
STUDIES (one Variable study)
  └─ STUDY_REVISIONS (v1 DRAFT → FINALIZED; later v2 if revised)
      ├─ APPRAISERS: 3 human identities
      ├─ SAMPLES: 10 part IDs
      └─ VARIABLE_READINGS: 10 parts × 3 appraisers × 3 trials = 90 rows
~~~

- While Draft, only entered measurement cells have rows (e.g. 5/90); empty is not numeric zero.
- Each filled cell is unique by **revision + appraiser + sample + trial**. Measurement stored as canonical decimal string, not rounded binary REAL.
- Service validates exactly 90 values, 10 part IDs, 3 appraisers and three trials per pair before Calculate; the database does not force completeness during Draft.
- After Finalized, underlying 90 rows cannot be updated/deleted; a Supervisor creates an **entirely new revision** and copies data with new revision-linked IDs.

### 3.2 Attribute study

~~~text
STUDIES (one Attribute study, not one per appraiser paper sheet)
  └─ STUDY_REVISIONS
      ├─ APPRAISERS: 3 human identities
      ├─ SAMPLES: 50 shared IDs
      ├─ ATTRIBUTE_DECISIONS: 50 × 3 × 2 = 300 OK/NG rows
      └─ ATTRIBUTE_REFERENCES: 50 separate known OK/NG labels
                              (Supervisor-only write/verify)
~~~

- All three appraisers share the **same** 50 sample identities. Source paper may be combined or three separate collection sheets without creating duplicate studies.
- Only Supervisor may create/edit/verify ATTRIBUTE_REFERENCES. Operator may view the **History listing for every study** and finalized permitted reports, but may edit only own Draft and must not see active blinded references.
- Before comparison with Known Reference, confirm blind assessment completion/closure. A verified reference is different from a merely populated cell.
- Pooled Kappa **per appraiser**, agreement and missed NG / false reject denominators are **derived**, not typed into a source table.
- Calculate requires 300 decisions, 50 verified references and allowed unblinding; missing verified data is not treated as OK.

### 3.3 Shared lifecycle

~~~text
Study (stable ID)
  → Revision v1 (DRAFT, input_version 0, incomplete allowed)
    → input edit increments input_version; prior calculation becomes STALE
    → Calculation Run (append-only, input_hash and method_version)
    → Review Request (PENDING, references a CURRENT run)
      → Supervisor Request Correction (DRAFT remains editable)
      → OR Supervisor Approve & Finalize (approved policy only)
          → Approval record + final PDF record + audit + FINALIZED status
  → Revision v2 (new Draft cloned from finalized snapshot, if needed)
~~~

Review status is not a substitute for the statistical outcome. **Review Required** can mean missing Quality acceptance rules or concerning indicators; it is never silently translated into pass.

## 4. Integrity constraints

| Rule | DB-enforceable part | Service/business validation |
| --- | --- | --- |
| Users and roles | PK, UNIQUE access_sub/email_norm, CHECK role | Validate Access JWT & current active user before every request |
| One study revision sequence | UNIQUE(study_id, revision_no) | Parent revision belongs to same study; no concurrent duplicate revision |
| One active Draft per study | SQLite partial UNIQUE index for DRAFT | Only Supervisor creates revision of a finalized study |
| Distinct appraisers | UNIQUE(revision_id,slot_no), UNIQUE(revision_id,normalized_name) | 3 unique names before Calculate |
| Distinct parts/samples | UNIQUE(revision_id,slot_no/normalized_label) | 10/50 appropriate to msa_type |
| Exactly one value per slot | UNIQUE(revision_id,appraiser_id,sample_id,trial_no) | Validate full 90/300 matrix, not merely 90/300 rows |
| Prevent cross-revision mixing | Composite FK to (revision_id,appraiser_id) and (revision_id,sample_id) | Enforce type and edit ownership |
| Decision / reference values | CHECK decision/known_label IN ('OK','NG') | Data-entry role and blind workflow |
| Reference one per sample | UNIQUE(revision_id,sample_id) | Only Supervisor verifies 50 references |
| Current result / approval | FK + immutable snapshots | Compare input_version and SHA-256 and method/policy versions |
| One final approval/report | UNIQUE approvals.revision_id and report_artifacts.approval_id | Supervisor-only approval, report uploaded/verified, finalization transaction |
| Auditable immutable history | FKs + reviewed trigger guards | No mutations after FINALIZED, no hard delete; operational backup |

**Concrete SQLite constraints to verify during migration design:**

~~~sql
-- Illustrative only; not an executable migration.
CREATE UNIQUE INDEX uq_one_draft_per_study
ON study_revisions(study_id)
WHERE lifecycle_status = 'DRAFT';

CREATE UNIQUE INDEX uq_variable_slot
ON variable_readings(revision_id, appraiser_id, sample_id, trial_no);

CREATE UNIQUE INDEX uq_attribute_slot
ON attribute_decisions(revision_id, appraiser_id, sample_id, trial_no);

CREATE UNIQUE INDEX uq_reference_slot
ON attribute_references(revision_id, sample_id);

CREATE UNIQUE INDEX uq_one_pending_review
ON review_requests(revision_id)
WHERE status = 'PENDING';

-- For every source row, FKs must also enforce same revision:
-- FOREIGN KEY (revision_id, appraiser_id)
--   REFERENCES appraisers(revision_id, id)
-- FOREIGN KEY (revision_id, sample_id)
--   REFERENCES samples(revision_id, id)
~~~

Make sure referenced composite pairs have matching UNIQUE constraints. D1 enforces foreign keys and supports SQLite schema/index semantics; verify resulting constraints and the exact Drizzle migration in a **local D1 test database** before deployment. Official references: [Cloudflare D1 foreign keys](https://developers.cloudflare.com/d1/sql-api/foreign-keys/), [D1 SQLite SQL statements and PRAGMA](https://developers.cloudflare.com/d1/sql-api/sql-statements/).

**No fragile latest_revision_id pointer:** get latest by descending revision_no with indexed study_id, while uniqueness and a one-Draft-only index enforce an unambiguous current editing revision.

## 5. Staleness, policy version and approval controls

- A new or edited source reading, appraiser/sample mapping, reference label/verification, or chosen policy increments the Draft input_version. Prior calculation runs stay append-only but no longer count as Current.
- Calculation run stores an input_sha256 over canonical, deterministically ordered source data and method/policy-relevant configuration. Hashing rules (normalization, ordering and field coverage) will be specified in API/engine contracts.
- Policy versions are append-only once APPROVED; they store Quality approval evidence and actual rules. In particular, **missed-NG and false-reject limits remain unapproved**, and the database must not invent default limits.
- A Review Request points to a specific CURRENT calculation run; the backend invalidates pending submissions when inputs change. Only one PENDING request per revision.
- Only an authenticated **Supervisor** can approve; role enforcement is on Worker routes, not merely frontend button visibility. Approval saves an approver identity/name/title snapshot and an authorized signature reference.
- Protect full Finalized revision rows (study metadata, appraisers, samples, decisions, references), approval and reports from later modification. Operational roles should never have a normal hard-delete route.
- Preserve UTC timestamps, method version, policy snapshot and immutable PDF byte hash so historical report outcomes do not drift when Quality policy, signatory or templates change.

### 5.1 Prevent R2/D1 partial finalization

R2 object storage and D1 are **not one distributed transaction**. Use a safe two-stage report issuance process:

1. Authenticate Supervisor; validate current input version/hash, policy approval, reference verification and signature/logo assets; reserve an idempotent approval intent.
2. Render and verify a candidate PDF; upload it under a **unique, private staging key** in R2 and record its SHA-256.
3. Within one **D1 atomic batch**, enforce version comparison at the database layer, link the exact calculation run + review request + approval + report_artifact and update revision to FINALIZED with an audit event. Abort the entire database transition on changed version/state, rather than merely trusting a previous SELECT.
4. Make the archived PDF downloadable through an authorized Worker route **only after** the DB state is finalized. Failed DB commit leaves no visible Finalized study; reconcile orphan staging objects safely.
5. Never overwrite an old PDF. New information or corrections require a Supervisor-created revision with a newly approved PDF.

This is a **design plan**, not proof the flow already works. The implementation gate must test concurrent edits, duplicate submit clicks, stale-policy/risk cases, PDF/R2 failures, retries, and DB commit failure.

## 6. Data dictionary and indexing

For the full 15-table column-by-column specification, types, optionality and ownership constraints, see **[Data Dictionary v0.1](data-dictionary-v0.1.md)**.

Recommended indexes beyond UNIQUE:
- studies(owner_user_id, created_at DESC), studies(msa_type, deleted_at)
- study_revisions(study_id, revision_no DESC), study_revisions(lifecycle_status, updated_at DESC)
- appraisers(revision_id), samples(revision_id)
- variable_readings(revision_id, appraiser_id, sample_id)
- attribute_decisions(revision_id, appraiser_id, sample_id)
- calculation_runs(revision_id, calculated_at DESC)
- review_requests(status, submitted_at DESC)
- approvals(approved_by_user_id, approved_at DESC)
- audit_events(study_id, happened_at DESC), audit_events(actor_user_id, happened_at DESC)

Avoid excessive indexing until realistic query plans and record volumes are measured. Use limit/pagination for all-history queries. Set a retention period with Ratama before implementing database and R2 cleanup.

## 7. Draft ERD acceptance checklist

| ID | Review/test | Expected |
| --- | --- | --- |
| DB-01 | Operator opens History | All studies list, other-owner Draft read-only; blind references not disclosed |
| DB-02 | Save Variable Draft with 5/90 readings | Allowed, 5 rows retained, no injected zeros |
| DB-03 | Store two readings with same revision/appraiser/sample/trial | UNIQUE prevents duplicate source slot |
| DB-04 | Reading points to sample from another revision | Composite FK blocks cross-revision corruption |
| DB-05 | Calculate Variable with 89/90 | Blocked; exact missing slot identified |
| DB-06 | Calculate Variable with complete 90 | One versioned result run, with independent golden arithmetic verification |
| DB-07 | Save partial Attribute with 57 decisions and 12 known labels | Allowed (Supervisor-only reference write), still Draft |
| DB-08 | Calculate Attribute with 300 decisions but 49 references | Kappa vs Reference blocked; no implicit OK |
| DB-09 | Operator requests Attribute reference API | Server denies, even for own study |
| DB-10 | Changing verified Attribute reference | Original result becomes stale; audit captures authorized Supervisor edit |
| DB-11 | Change input after submitted review | Pending request invalidated; re-calculate and re-submit needed |
| DB-12 | Two concurrent Supervisor Finalize requests | At most one approval and one final report, exactly one FINALIZED transition |
| DB-13 | PDF created in R2 but DB commit rejected | No visible Finalized state; staging object cleanup/reconciliation |
| DB-14 | Ratama Quality policy unapproved or missing | Calculated metrics retained; cannot approve an invented PASS |
| DB-15 | Create revision of Finalized study | v1 source/approval/report unchanged; v2 receives independent rows |
| DB-16 | User is deactivated after historic approval | Approval snapshot still records actual past signer; new approvals blocked |
| DB-17 | Attempt to mutate finalized reading/reference/report | Rejected by backend and database-level protections |
| DB-18 | User accesses archived PDF | Read authorized stored R2 bytes, matching archived SHA-256 |

**These are specification acceptance scenarios, not executed tests.**

## 8. Known open decisions

1. Ratama Quality sign-off for Attribute missed NG, false reject and whole-study outcome criteria; Variable composite acceptance/exception handling.
2. Precise signature/branding asset custody and whether the official report requires scanned or regulated e-signature.
3. Access OTP login replacing historic email/password form.
4. D1 migration and trigger feasibility; transaction guards, backup/restore and indexed History under realistic loads.
5. How alternative part/appraiser/trial designs would be versioned and validated beyond current presets.
6. PDF rendering and R2 lifecycle proof of concept before deciding production retention/issuance.
7. Future multi-client/SPC scope is explicitly excluded.

**Review verdict:** The ERD is coherent for the agreed single-client MVP and consistent with DEC-027 to DEC-033, but should be **reviewed before migration code is produced**, not treated as a FROZEN production schema.
