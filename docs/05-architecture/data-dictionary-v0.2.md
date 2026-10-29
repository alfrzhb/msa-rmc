# RMC MSA — Database Data Dictionary v0.2

**Status:** REVISED DRAFT / NOT FROZEN (9 audit findings addressed in design)  
**Date:** 29 October 2026  
**Engine:** Cloudflare D1 (SQLite), with Drizzle ORM as the proposed schema/migrations layer  
**Companion:** [Database ERD v0.2](erd-database-v0.2.md) | [Mermaid source](erd-v0.2.mmd) | [v0.1 archive](data-dictionary-v0.1.md) | [Tech Stack Decision](tech-stack-decision-v1.0.md)

**v0.2 is the normative candidate dictionary.** The v0.1 file remains a historical design; no D1 migrations were executed. 15 tables remain, with added cross-table constraints and audit/lock fields.

## Conventions

- **PK** primary key; **FK** foreign key; **UQ** unique constraint; **NN** NOT NULL; **C** candidate CHECK constraint; **A** immutable after finalized/approved; **N** nullable.
- IDs: application-generated, collision-resistant TEXT identifiers (UUIDv7 candidate). In SQLite migrations, explicitly use TEXT PRIMARY KEY NOT NULL and UNIQUE indexes where required.
- Timestamps: UTC ISO 8601 with fixed representation in TEXT; render Asia/Jakarta in the UI. Study calendar date is YYYY-MM-DD TEXT.
- Boolean: INTEGER 0/1 with CHECK. Category enums: TEXT + CHECK. JSON snapshots: TEXT, validated with json_valid and a versioned Zod schema.
- Decimal measurement: **canonical decimal string stored in TEXT**, parsed with controlled validation in calculation module. Do not round source readings or rely on SQLite REAL as the persisted source of truth.
- Rows for readings/decisions/references are **sparse while Draft**: absent row = not entered, never implicit 0, OK or NG. Calculate checks exact design completeness.
- Parent and child revision consistency requires **composite foreign keys** (revision_id + appraiser_id/sample_id); a simple foreign key on ID alone would allow accidentally mixing revisions.
- Hard deletion of study history, audit, approved policies and final artifacts is not exposed in application UI. All sensitive media remains in private R2.

## Identity and studies

### 1. users

| Column | Storage / constraint | Meaning |
| --- | --- | --- |
| id | TEXT PK NN | Internal immutable user ID |
| access_sub | TEXT UQ NN | Verified Cloudflare Access subject; primary stable identity mapping |
| email_norm | TEXT UQ NN | Normalized work email, may change by controlled admin process |
| display_name | TEXT NN | Employee display name |
| role | TEXT NN C | ADMIN, SUPERVISOR or OPERATOR |
| is_active | INTEGER NN C | 1 active, 0 disabled |
| created_at, updated_at | TEXT NN | UTC audit |

Use only verified Access JWT claims to resolve access_sub. No local password or password-hash column in this stack.

### 2. studies

| Column | Storage / constraint | Meaning |
| --- | --- | --- |
| id | TEXT PK NN | Persistent study ID across revisions |
| study_code | TEXT UQ NN | Human-readable stable reference, generated and collision-resistant |
| msa_type | TEXT NN C | VARIABLE or ATTRIBUTE; immutable after source input |
| owner_user_id | TEXT FK(users) NN | Responsible study owner / editor |
| created_by_user_id | TEXT FK(users) NN | Initial authenticated creator |
| created_at, updated_at | TEXT NN | Server event times |
| deleted_at, deleted_by_user_id | TEXT N, TEXT FK(users) N | Supervisor soft-delete / restore audit |

**Do not add current_revision_id to studies in v0.1**: it introduces a circular FK and a synchronization problem. Query latest revision by (study_id, revision_no DESC), and allow only one Draft at a time per study.

### 3. study_revisions

| Column | Storage / constraint | Meaning |
| --- | --- | --- |
| id | TEXT PK NN; UQ(study_id,id) | Each revision identity; composite self-FK target |
| study_id | TEXT FK(studies) NN | Parent study |
| revision_no | INTEGER NN UQ(study_id,revision_no) | 1,2,3…; monotonic, no reuse |
| parent_revision_id | TEXT N; composite FK (study_id,parent_revision_id) | Prior FINALIZED or SUPERSEDED revision **in the same study** |
| lifecycle_status | TEXT NN C | **DRAFT / FINALIZED / SUPERSEDED**. SUPERSEDED archives a compromised/disclosed Draft without a final approval |
| input_version | INTEGER NN C, starts 0 | Increment on **any change affecting input, reference or selected policy** |
| selected_policy_id | TEXT FK(judgment_policies) N | **NEW:** explicit versioned Quality policy selected; changes invalidate prior calculations |
| current_input_sha256 | TEXT N | **NEW:** canonical source digest set by Calculate; NULL after every calculation-affecting edit; compare with submitted run at Finalize |
| study_name, study_date | TEXT NN for Calculate | User-visible title, local business date |
| part_name, part_number, model | TEXT | Product identity; part_name required for Calculate |
| characteristic | TEXT | Measured/inspected characteristic, required for Calculate |
| process_line, study_context, performed_by, notes | TEXT N | Context; optional unless local policy requires |
| gage_name, gage_number, gage_type, unit | TEXT N | Variable-specific; gage_name and unit required for Variable Calculate |
| nominal_text, lsl_text, usl_text | TEXT N | Canonical decimals; if limits given, require both and USL>LSL |
| gage_resolution_text, calibration_reference | TEXT N | Instrument context |
| design_part_count, design_appraiser_count, design_trial_count | INTEGER NN | Versioned fixed method preset; Variable 10/3/3 or Attribute 50/3/2 |
| reference_source | TEXT N | Attribute reference provenance; Supervisor context |
| attribute_lock_status | TEXT N C | **NEW:** ATTRIBUTE=OPEN / LOCKED / DISCLOSED; VARIABLE=NULL |
| blind_closed_at, blind_closed_by_user_id | TEXT N, TEXT FK(users) N | Supervisor closes all blind Attribute decisions, status becomes LOCKED |
| reference_disclosed_at, reference_disclosed_by_user_id | TEXT N, TEXT FK(users) N | **NEW:** irreversible disclosed reference-derived results, status DISCLOSED |
| created_by_user_id, created_at, updated_at | FK(users), TEXT NN | Draft authorship and audit |
| finalized_at | TEXT N | Set only by valid Supervisor Approve & Finalize |
| superseded_at, superseded_by_user_id, supersede_reason | TEXT N, TEXT FK(users) N, TEXT N | **NEW:** reasoned Supervisor archival of a disclosed Draft requiring recollection |

**Pre-calculation validation lives in the service**; sparse Draft metadata is supported. Partial UNIQUE(study_id) WHERE lifecycle_status='DRAFT' allows at most one active Draft; FINALIZED and SUPERSEDED source must be immutable. Any successor revision gets **independent** appraiser, sample and source IDs. After DISCLOSED, correction needs Supervisor SUPERSEDE + new independent blind collection, not reopening the prior answers. Only Supervisor may transition OPEN→LOCKED→DISCLOSED; before DISCLOSED a reasoned correction/reopen is possible and invalidates all runs. No results derived from Known Reference may reach Operator while input remains editable. Selected policy must match msa_type and be APPROVED for final disposition; exploratory runs with no approved policy remain REVIEW_REQUIRED.

## Inputs

### 4. appraisers

| Column | Storage / constraint | Meaning |
| --- | --- | --- |
| id | TEXT PK NN | Person doing measurements; not a platform login |
| revision_id | TEXT FK(study_revisions) NN | Parent revision |
| slot_no | INTEGER NN C UQ(revision_id,slot_no) | 1..3 for initial preset |
| display_name | TEXT NN | Human-readable name |
| normalized_name | TEXT NN UQ(revision_id,normalized_name) | Prevent case/space duplicate appraisers |
| employee_code | TEXT N | Optional local staff identifier |

Add UQ(revision_id,id) to support composite FK from readings and decisions.

### 5. samples

| Column | Storage / constraint | Meaning |
| --- | --- | --- |
| id | TEXT PK NN | Stable **within revision**, new ID when creating a revision |
| revision_id | TEXT FK(study_revisions) NN | Parent revision |
| slot_no | INTEGER NN C UQ(revision_id,slot_no) | Variable 1..10; Attribute 1..50 |
| sample_label | TEXT NN | User-friendly part/sample identifier |
| normalized_label | TEXT NN UQ(revision_id,normalized_label) | No duplicate sample labels |

Add UQ(revision_id,id) to support composite FK from reading/decision/reference rows.

### 6. variable_readings

| Column | Storage / constraint | Meaning |
| --- | --- | --- |
| id | TEXT PK NN | Individual recorded reading |
| revision_id | TEXT FK(study_revisions) NN | Parent revision |
| appraiser_id | TEXT FK(appraisers) NN | Must belong to same revision |
| sample_id | TEXT FK(samples) NN | Must belong to same revision |
| trial_no | INTEGER NN C | 1,2,3 in validated Variable preset |
| value_decimal | TEXT NN | Canonical decimal, no NaN/Infinity |
| entered_by_user_id, entered_at | FK(users), TEXT NN | Audit actor separate from source appraiser |
| updated_by_user_id, updated_at | FK(users), TEXT | Attribution for corrections |

**UNIQUE(revision_id,appraiser_id,sample_id,trial_no)**, plus composite foreign keys (revision_id,appraiser_id) and (revision_id,sample_id). No missing-value row should be created with a fabricated zero.

### 7. attribute_decisions

| Column | Storage / constraint | Meaning |
| --- | --- | --- |
| id | TEXT PK NN | Individual classification |
| revision_id | TEXT FK(study_revisions) NN | Parent revision |
| appraiser_id | TEXT FK(appraisers) NN | Must belong to same revision |
| sample_id | TEXT FK(samples) NN | Must belong to same revision |
| trial_no | INTEGER NN C | 1 or 2 |
| decision | TEXT NN C | OK or NG only |
| entered_by_user_id, entered_at | FK(users), TEXT NN | Actor who transcribed decision |
| updated_by_user_id, updated_at | FK(users), TEXT | Controlled edit attribution |

**UNIQUE(revision_id,appraiser_id,sample_id,trial_no)** and same composite-revision FK pattern. Appraiser answer remains hidden from the Known Reference comparison until blind collection is locked.

### 8. attribute_references

| Column | Storage / constraint | Meaning |
| --- | --- | --- |
| id | TEXT PK NN | One official known label per sample |
| revision_id | TEXT FK(study_revisions) NN | Parent Attribute revision |
| sample_id | TEXT FK(samples) NN | Must belong to same revision |
| known_label | TEXT NN C | OK or NG, never majority-vote autofill |
| source_note | TEXT N | Optional evidence/provenance per reference |
| verified_by_user_id | TEXT FK(users) N | Supervisor user; blank until verified |
| verified_at | TEXT N | Time of Supervisor verification |
| updated_by_user_id, updated_at | FK(users), TEXT | Supervisor-only edit audit |

**UNIQUE(revision_id,sample_id)**. Verification is explicit, **not inferred** from the presence of a label. Permission to read/write this table is **Supervisor-only in active blind studies**, even if Operators can see all History.

## Calculations and approval policy

### 9. judgment_policies

| Column | Storage / constraint | Meaning |
| --- | --- | --- |
| id | TEXT PK NN | Immutable policy-version ID |
| policy_key | TEXT NN | Logical family such as VARIABLE_RATAMA / ATTRIBUTE_RATAMA |
| version_no | INTEGER NN UQ(policy_key,version_no) | Numbered policy version |
| status | TEXT NN C | DRAFT, APPROVED, RETIRED |
| applicability | TEXT NN | VARIABLE / ATTRIBUTE |
| rules_json | TEXT NN C | Versioned approved threshold definitions, not formula code |
| approved_by_user_id, approved_at | FK(users) N, TEXT N | Who records approved Quality policy |
| approval_evidence | TEXT N | Actual Quality/customer sign-off reference; required to APPROVE |
| created_by_user_id, created_at | FK(users) NN, TEXT NN | Immutable creation record |

Only APPROVED policy versions may authorize automatic overall acceptance. **No numeric missed-NG/false-reject limits are assumed until Ratama Quality supplies them**. Retiring a version must not change historical snapshots.

### 10. calculation_runs

| Column | Storage / constraint | Meaning |
| --- | --- | --- |
| id | TEXT PK NN; UQ(revision_id,id) | One independent run; composite review/approval FK target |
| revision_id | TEXT FK(study_revisions) NN | Source revision |
| input_version | INTEGER NN | Revision version at time of calculation |
| input_sha256 | TEXT NN | Hash of normalized source & method/policy-relevant input |
| method_key, method_version | TEXT NN | Corrected Variable A&R or pooled Attribute Kappa implementation version |
| policy_id | TEXT FK(judgment_policies) N | May be null (metrics without approved Quality policy) |
| policy_snapshot_json | TEXT N C | Exact evaluated policy at calculation time |
| results_json | TEXT NN C | Versioned derived statistics and explicit Not Calculable/warnings |
| calculated_band | TEXT N | Mathematical band distinct from final Quality disposition |
| overall_disposition | TEXT NN C | REVIEW_REQUIRED or policy-authorized decision |
| calculated_by_user_id, calculated_at | FK(users) NN, TEXT NN | Provenance |

Each run is **append-only**. Do not mutate a run when Draft input changes. A run is Current iff input_version and hash still match its parent revision/normalized input, and required reference-blinding conditions are satisfied.

### 11. review_requests

| Column | Storage / constraint | Meaning |
| --- | --- | --- |
| id | TEXT PK NN; UQ(revision_id,id); UQ(revision_id,id,calculation_run_id) | Submitted attempt; composite approval FK target |
| revision_id | TEXT FK(study_revisions) NN | Current Draft being reviewed |
| calculation_run_id | TEXT NN; composite FK(revision_id,calculation_run_id) | Must reference a run belonging to this revision |
| request_no | INTEGER NN UQ(revision_id,request_no) | Increasing attempt number |
| status | TEXT NN C | PENDING, NEEDS_CORRECTION, CANCELLED, APPROVED |
| submitted_by_user_id, submitted_at | FK(users) NN, TEXT NN | Owner/Supervisor who submitted |
| reviewed_by_user_id, reviewed_at | FK(users) N, TEXT N | Supervisor decision actor |
| correction_reason | TEXT N | Explanation when correction requested |

Partial UNIQUE(revision_id) WHERE status='PENDING'. Changing an input/references/policy invalidates PENDING request and calculated result freshness; maintain the status transition and audit together. A request that was approved must link exactly one approval.

### 12. approvals

| Column | Storage / constraint | Meaning |
| --- | --- | --- |
| id | TEXT PK NN | Audit identity for one finalized revision |
| idempotency_key | TEXT UQ NN | Replay-safe finalization intent/revision key |
| revision_id | TEXT FK(study_revisions) NN UQ | A finalized revision has at most one approval |
| review_request_id | TEXT NN UQ | Composite FK with revision and calculation_run_id to the exact review request |
| calculation_run_id | TEXT NN; composite FK(revision_id,calculation_run_id) | Must match the submitted request and revision |
| approved_by_user_id | TEXT FK(users) NN | Authenticated Supervisor |
| approved_at | TEXT NN | Exact server approval time |
| approver_name_snapshot, approver_title_snapshot | TEXT NN | Immutable identity for report |
| quality_disposition | TEXT NN | Exact policy-authorized final result |
| policy_snapshot_json | TEXT NN C | Versioned policy and approval evidence at Finalize |
| signature_asset_id | TEXT NN; composite FK(signature_asset_id,approved_by_user_id) | Must reference SIGNATURE asset belonging to actual approver |
| approval_note | TEXT N | Required for authorized exception disposition |

Approval must be recorded **atomically with** Finalized revision status, a report_artifacts row, and audit event; use DB-enforced optimistic version guard, not only a pre-read. Approval being present does not mean any arbitrary unsupported risk result may become 'OK'.

### 13. report_artifacts

| Column | Storage / constraint | Meaning |
| --- | --- | --- |
| id | TEXT PK NN | Individual immutable final PDF |
| approval_id | TEXT FK(approvals) NN UQ | Exactly one official report per approved revision |
| r2_key | TEXT NN UQ | Private R2 immutable object name (revision + content hash) |
| sha256_hex | TEXT NN | Actual PDF byte hash |
| content_type | TEXT NN C | application/pdf |
| bytes_size | INTEGER NN C | PDF byte length >0 |
| logo_asset_id | TEXT FK(brand_assets) NN | Logo asset used when report was generated |
| template_version | TEXT NN | Version of print template |
| issued_at | TEXT NN | Final report archival time |

A PDF is built and uploaded to an **unpublished staging key** first; then an approval transaction commits the report metadata and Final state. On failure, the final DB state is not asserted; orphan staging objects are reconciled. Return stored PDF bytes, never a regenerated version using later policy/assets.

## Assets and audit

### 14. brand_assets

| Column | Storage / constraint | Meaning |
| --- | --- | --- |
| id | TEXT PK NN; UQ(id,signer_user_id) | Logo/signature asset; unique signer tuple for composite FK |
| asset_kind | TEXT NN C | LOGO or SIGNATURE |
| r2_key | TEXT NN UQ | Private R2 object key |
| signer_user_id | TEXT FK(users) N | CHECK: SIGNATURE requires signer, LOGO requires NULL; paired FK prevents wrong signer |
| content_sha256 | TEXT NN | Asset integrity |
| content_type, bytes_size | TEXT NN, INTEGER NN | Validated MIME, bounded bytes |
| is_active | INTEGER NN C | Can use for new approval; old artifacts reference prior asset |
| uploaded_by_user_id, uploaded_at | FK(users) NN, TEXT NN | Authorized asset provenance |

A signature image is **not** a cryptographically certified signature. Only a Supervisor's authenticated Approve & Finalize action can associate a signer-authorized signature with official report.

### 15. audit_events

| Column | Storage / constraint | Meaning |
| --- | --- | --- |
| id | TEXT PK NN | Append-only event identity |
| study_id | TEXT FK(studies) N | Study context; NULL for global account/policy events |
| revision_id | TEXT FK(study_revisions) N | Optional revision context; must belong to study when study_id is present |
| actor_kind | TEXT NN C | USER or SYSTEM |
| actor_user_id | TEXT FK(users) N | CHECK user present iff actor_kind=USER; otherwise NULL |
| action_type | TEXT NN | STUDY_CREATED, CELL_CHANGED, REFERENCE_VERIFIED, SUBMITTED, CORRECTION_REQUESTED, POLICY_APPROVED, FINALIZED, REVISION_CREATED, etc. |
| entity_type, entity_id, field_path | TEXT N | Precise target cell/field attribution |
| before_json, after_json | TEXT N C | **NEW** exact minimally necessary old/new values for cell corrections; restricted access |
| sensitivity | TEXT NN C | STUDY, RESTRICTED_REFERENCE, SECURITY; no Operator access to reference audit |
| reason | TEXT N | Required for unlock, correction, exceptional disposition, SUPERSEDED |
| event_json | TEXT N C | Optional structured nonsecret metadata (never JWT or credential) |
| happened_at | TEXT NN | UTC append time |

Append-only via database permissions/application protections and reviewed triggers; use soft delete for application entities and protected backups to avoid unilateral destruction of the historical record.

## Constraints that cannot be expressed by a single simple CHECK

These require deterministic application validation and/or reviewed SQLite triggers (not yet implemented):

1. study.msa_type must match which of variable_readings / attribute_decisions / attribute_references can be written.
2. An appraiser/sample must belong to the **same revision** as a reading/decision/reference, including through composite FKs.
3. Exactly 3 appraisers, 10 Variable samples × 3 trials or 50 Attribute samples × 2 trials only when **Calculate**, never when saving a partial Draft.
4. Attribute reference verification only by an active Supervisor; verified references and closed blind trials required before comparing outcomes to reference.
5. Any edit affecting calculation increments input_version and invalidates open approval submissions.
6. Current run hash/version, approved policy snapshot, authorized Supervisor signature/logo, valid review request and immutable PDF bytes are necessary to approve/finalize.
7. All rows of a Finalized revision, its approval and report linkage are immutable; create a new revision instead.
8. Prevent inconsistent revision parentage and more than one unfinished Draft per study.

**Schema/testing gate:** Implement constraints with Drizzle-generated D1-compatible migrations, plus explicit hand-reviewed SQL where necessary. Do not publish this dictionary as executed migrations or claim database trigger coverage before local D1/SQLite tests pass.


## v0.2 binding rules and changes that supersede v0.1 descriptions

### Cross-revision safety (H1/H2)

- study_revisions has UNIQUE(study_id,id) so a composite self-FK (study_id,parent_revision_id) cannot point to a different study.
- appraisers, samples and calculation_runs require UNIQUE(revision_id,id). review_requests requires UNIQUE(revision_id,id) **and** UNIQUE(revision_id,id,calculation_run_id).
- variable_readings and attribute_decisions require composite FKs (revision_id,appraiser_id) → appraisers(revision_id,id) and (revision_id,sample_id) → samples(revision_id,id). attribute_references requires (revision_id,sample_id) → samples(revision_id,id).
- review_requests FK (revision_id,calculation_run_id) → calculation_runs(revision_id,id).
- approvals FK **(revision_id,review_request_id,calculation_run_id)** → review_requests(revision_id,id,calculation_run_id), and FK (revision_id,calculation_run_id) → calculation_runs(revision_id,id). Thus an approval cannot refer to a review of a different calculation or revision.
- Preserve UNIQUE approvals(revision_id), approvals(review_request_id) and report_artifacts(approval_id). Finalized revisions must have exactly one committed report. The ORM relationship alone does not guarantee it.

### Blind Attribute closure (H4/H6/H7)

- Attribute status **OPEN** permits source answer editing, **LOCKED** prohibits answer edits, **DISCLOSED** records irreversible first exposure of reference-dependent results. The API must not expose per-appraiser Kappa/accuracy/error, confusion matrix or any reference-derived metrics before authorized disclosure.
- Only Supervisor can lock 300/300 independent answers, verify 50 official references and disclose derived results. A reasoned Supervisor reopen is permissible **only before DISCLOSED** with version increment and audit; after disclosure, mark the old revision SUPERSEDED and collect new data under a new revision. Do not relabel formerly exposed answers as blind.
- Changing an Attribute known_label must atomically clear its verified_by_user_id and verified_at; a reviewed BEFORE UPDATE trigger should reject retaining verification when the label changes. Verification must be renewed by Supervisor. Formula input hash and version become stale after any such change.

### Policy / signer / audit (H1/H5/H8)

- selected_policy_id in study_revisions is explicitly nullable while Draft. Selecting a different version increments input_version; only an active approved Quality policy with applicable MSA type and actual evidence may authorize a final acceptance result. The calculation run retains its own snapshot and policy_id equal to the selected version at that time.
- **brand_assets** CHECK enforces asset_kind='LOGO' iff signer_user_id IS NULL and asset_kind='SIGNATURE' iff signer_user_id IS NOT NULL. approvals composite FK (signature_asset_id,approved_by_user_id) → brand_assets(id,signer_user_id) ensures the signatory owns the SIGNATURE. report_artifacts.logo_asset_id is checked as LOGO by service + reviewed trigger; referenced assets may not be overwritten after official issuance.
- Record actual before/after changed measurement values in restricted audit, and change actor/time/reason. For reference labels the event sensitivity is RESTRICTED_REFERENCE and redacted from all non-Supervisor API responses. Audit changes are appended within the same D1 batch as the input mutation/version bump.

### Finalization and concurrency (H3/H5)

- D1 is transactional for **batch()** but R2 is an external object store. First upload & validate unique private PDF bytes under a nonpublic R2 key. Then run **one** D1 batch for approval, report link, final state and audit.
- A guarded **BEFORE INSERT ON approvals trigger** must reject if the referenced revision is not DRAFT, current input_version/hash differs from the submitted calculation, request is not PENDING, policy is unapproved/inapplicable, or actor lacks required Supervisor role. D1 operation order: (1) insert approval (guard), (2) insert verified report_artifacts, (3) update revision to FINALIZED, with **AFTER UPDATE status-to-FINALIZED trigger** requiring approval plus report, (4) update request APPROVED and insert audit. Every statement is within the same batch. Test exact trigger ordering and guarded state checks in an actual D1 migration.
- Do **not** rely on UPDATE matching zero rows to automatically fail. The approval trigger must fail with a SQL error (e.g. RAISE(ABORT) in a reviewed trigger) for stale version/state, so the entire batch rolls back. UNIQUE constraints and an idempotency key derived from revision+run+request prevent duplicate Finalize. An R2 upload without a DB commit is an **orphan**, not a visible final report; reconcile safely.
- Reject UPDATE/DELETE of finalized/superseded source rows, approved immutable policy content, approvals, report artifacts and audit events using service permissions and reviewed D1 triggers. Mutation of policy status for retirement may be allowed but must not rewrite snapshots.

### Additional column-level v0.2 commitments

- judgment_policies adds **quality_authority TEXT N** and **schema_version INTEGER NN** to distinguish authentic Quality signoff from a Supervisor recording it; once APPROVED, rules and approval evidence are immutable.
- calculation_runs adds **results_schema_version INTEGER NN** and stores method_key / method_version, input_version / input_sha256 and policy snapshot as one immutable unit.
- approvals adds **policy_schema_version INTEGER NN** and requires verified policy evidence plus an immutable name/title snapshot; quality_disposition must be supported by policy rules (no invented PASS).
- report_artifacts.logo_asset_id must point to a LOGO and report SHA-256 must match actual R2 PDF bytes. R2 key is unique, private and not overwritten; template_version immutable.
- study_revisions design counts are initially fixed to validated **Variable 10×3×3 (90)** and **Attribute 50×3×2 (300 + 50 references)**. Future flexible designs need separately versioned statistical constants and QA, not just editable integers.

### Schema review gate

All above rules are **design contracts**, not verified SQL migrations. Test partial unique indexes, composite FKs, JSON CHECKs, trigger failures/rollback, reference disclosure, immutable history, duplicate Finalize and uploaded-R2/failed-D1 reconciliation before declaring production readiness. See [ERD v0.2](erd-database-v0.2.md).

**Authoritative digest invariant:** Every source, appraiser/sample configuration, selected-policy or verification edit atomically increments revision.input_version and clears current_input_sha256. The Calculate service reads a consistent source snapshot, hashes canonical ordered values, and conditionally stores digest only if input_version did not change; it inserts the corresponding immutable calculation run. A Finalize guard compares both input_version and current_input_sha256 to the submitted run. This is a **service plus trigger requirement**, not an automatic property of SQLite.
