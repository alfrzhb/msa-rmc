# RMC MSA — Field & Screen Specification v0.1

**Status:** DRAFT — FOR REVIEW, NOT FROZEN  
**Tanggal:** 23 Oktober 2026  
**Produk:** RMC MSA Analysis Tool (MVP, satu perusahaan klien)  
**Basis:** Requirements Analysis v0.3; Ratama Variable/Attribute XLSX; Ratama PPT; historical Field & Screen v1.0 (reference only); Calculation Decisions v0.3; Attribute Study Input Workflow v0.1; Attribute Judgment Policy v0.1.

> Tujuan: menetapkan screen, field, aksi, validasi, dan hak akses untuk MVP sebelum membuat wireframe/implementasi. Isi form diturunkan dari *makna data* Ratama, bukan dari koordinat sel Excel. Keputusan yang belum disetujui sebagai kebijakan Quality harus dinyatakan OPEN, bukan diasumsikan FROZEN.

**Perubahan dari dokumen historis:** Field & Screen v1.0 lama tidak memuat role/login dan pengaturan judgment policy. Dokumen rebuild ini mengikuti keputusan terkini tentang Admin/Supervisor/Operator, study audit, dan kebijakan Attribute—tanpa menganggap freeze historis sebagai persetujuan baru.

## 1. Prinsip desain

1. **Sederhana:** Login → Dashboard/History → Create Study → Setup → Input → Calculate & Review → Finalize → Reprint PDF.
2. **Satu studi, satu sumber data:** Satu studi mencakup seluruh appraiser dan referensi; bukan satu studi per kertas atau appraiser.
3. **Preset awal:** Variable 10 parts × 3 appraisers × 3 trials; Attribute 50 samples × 3 appraisers × 2 trials. Ukuran preset hanya dapat diganti pada versi metode/desain yang telah divalidasi.
4. **Excel-friendly input:** tab per appraiser, keyboard navigation, paste rectangular dari Excel, validasi cell-level; tidak ada XLSX file import dalam MVP.
5. **Save Draft anytime:** data parsial boleh disimpan. Tombol **Calculate** hanya bekerja setelah semua input wajib untuk kalkulasi valid; jangan mengunci navigasi hanya karena input belum lengkap.
6. **Peran & kepemilikan:** Operator membuat/mengedit studi miliknya, Supervisor mengelola studi dan revisi, Admin mengelola akun. Hak akses pada server wajib sama dengan yang ditampilkan UI.
7. **Audit & reproduktibilitas:** studinya menyimpan identitas appraiser (orang yang mengukur) terpisah dari akun penginput; versi formula dan policy, snapshot hasil final, audit log, serta histori revisi.
8. **Perhitungan bukan formulir bebas:** EV/AV/GRR dan Kappa dihitung otomatis, bukan field yang dapat diketik. Formula tidak bisa diedit pengguna.
9. **Referensi Attribute:** hasil referensi OK/NG tidak boleh diperlihatkan kepada appraiser selama pengukuran buta berlangsung. Pengaturan siapa yang boleh menginput/mengesahkan referensi tetap terbuka; implementasi awal direkomendasikan oleh Supervisor.
10. **Kebijakan keputusan:** Standard Preset + Configurable **Approved** Policy. Hanya Supervisor yang memilih/mengelola kebijakan yang sudah memiliki persetujuan Quality. Tanpa kriteria wajib yang sah, laporan menampilkan perhitungan dan **Review Required**, bukan status kelulusan otomatis.

## 2. Screen inventory dan navigasi

| ID | Screen | Tujuan utama | Peran |
| --- | --- | --- | --- |
| S01 | Login | Autentikasi individual | Public (login) |
| S02 | Dashboard | Ringkasan studi dan pintasan | Operator, Supervisor (sesuai scope) |
| S03 | MSA History | Cari/filter/buka studi | Operator, Supervisor (sesuai scope) |
| S04 | New MSA — Type Selection | Pilih Variable / Attribute | Operator, Supervisor |
| S05 | Study Setup | Metadata dan konteks pengukuran | Pemilik studi/Supervisor |
| S06 | Appraisers & Samples | Isi 3 appraiser + ID part/sampel | Pemilik studi/Supervisor |
| S07 | Variable Measurement Input | 90 angka, tiga tab appraiser | Pemilik studi/Supervisor |
| S08 | Attribute Assessment Input | 300 OK/NG, tiga tab appraiser + reference terpisah | Pemilik studi/Supervisor; reference: kontrol tambahan |
| S09 | Calculation Results & Review | Hasil, peringatan, keputusan dan finalisasi | Pemilik studi/Supervisor, tergantung policy |
| S10 | Study Detail & Revisions | Metadata, raw data read-only, audit, versi, revisi | Sesuai hak baca dan revisi |
| S11 | Report Preview / PDF | Pratinjau, ekspor dan cetak | Sesuai hak baca |
| S12 | User Management | Kelola akun dan role | Admin |
| S13 | Judgment Policy | Pilih, lihat, siapkan versi kebijakan yang disetujui | Supervisor |

### Navigation map

```text
S01 Login
  └─ S02 Dashboard
       ├─ S03 History ───────────────┐
       ├─ S04 Create New MSA        │
       │    └─ S05 Study Setup      │
       │         └─ S06 Appraisers & Samples
       │              ├─ S07 Variable Input
       │              └─ S08 Attribute Input (+ restricted Reference)
       │                   └─ S09 Calculate / Results / Review
       │                        └─ Finalize
       └─ S10 Study Detail ◀────────┘
            ├─ S11 Report / PDF
            └─ Revision history
S12 User Management (Admin)
S13 Judgment Policy (Supervisor)
```

**Reuse:** S05/S06/S09/S10/S11 are shared components/flows with type-specific fields and result panels. UI may implement S05–S09 as a wizard with progress steps; they do not have to be five unrelated pages.

## 3. Data dictionary dan ownership

### 3.1 Common Study metadata — S05

**Legend:** R = required before **Calculate**; O = optional; A = automatic/read-only; C = conditional. Draft may be stored with incomplete R fields.

| Field key | UI label | Type | R/O/A | Source / rule |
| --- | --- | --- | --- | --- |
| `study_id` | Study ID | Generated code | A | Unique, server-generated; never manually editable |
| `study_name` | Study Name | Text | R | Suggested auto-name; may edit; non-empty, bounded length |
| `msa_type` | MSA Type | Variable / Attribute | A | Chosen S04; immutable once data exist |
| `study_date` | Study Date | Date | R | Default current local business date; editable |
| `part_name` | Part / Product Name | Text | R | From both Ratama workbooks |
| `part_number` | Part No. | Text | O | From Attribute sheet; optional for Variable |
| `model` | Model | Text | O | From Attribute sheet |
| `characteristic` | Characteristic | Text | R | e.g., measured characteristic or visual defect class |
| `process_line` | Process / Line | Text | O | Descriptive only, **not** a hierarchy module |
| `study_context` | Measurement / Inspection Method | Text | O | Helpful provenance, especially Attribute |
| `performed_by` | Performed By / Coordinator | Text | O | Human study coordinator, distinct from login user |
| `notes` | Notes | Multiline | O | Instructions/observations, no formula/control language |
| `owner_user_id` | Study Owner | From account | A | Person who created/owns editable study |
| `created_by`, `created_at` | Created By / At | Audit | A | Server-side |
| `updated_by`, `updated_at` | Last Updated | Audit | A | Server-side |
| `lifecycle_status` | Study Status | Draft / Finalized / Revision draft | A | Server-managed |
| `revision_no`, `parent_revision_id` | Revision | Version | A | Immutable final versions; lineage maintained |
| `calculation_method_version` | Method & Version | Read-only | A | Snapshot at calculation and finalization |

**Unique labels**: Part/sample identifiers must be unique inside a study. Appraiser names must be nonempty and distinct after normalization, but should not be forced to match platform users.

### 3.2 Variable study-only metadata — S05

| Field key | UI label | Type | R/O/A | Rule |
| --- | --- | --- | --- | --- |
| `gage_name` | Gage Name | Text | R | Source: Ratama Variable sheet |
| `gage_number` | Gage No. / Asset ID | Text | O | Source label |
| `gage_type` | Gage Type | Text | O | Source label |
| `unit` | Unit | Select/defined text | R | One consistent unit for measurements and LSL/USL; e.g., mm |
| `nominal_value` | Nominal / Target | Decimal | O | Optional, only if explicitly provided |
| `lsl` | Lower Specification Limit | Decimal | C | Both LSL & USL needed for %Tolerance; may be absent for %Study Var |
| `usl` | Upper Specification Limit | Decimal | C | Must satisfy USL > LSL if supplied |
| `specification_note` | Display Specification | Computed/text note | A/O | Format LSL/USL/nominal for report, not source of arithmetic |
| `gage_resolution` | Instrument Resolution | Positive decimal | O (recommended) | Include validation/warning if measured precision appears inconsistent |
| `calibration_reference` | Calibration Ref. / Status | Text | O (recommended) | Documentary context, not proof of calibration |
| `part_count`, `appraiser_count`, `trial_count` | Study Design | Read-only preset | A | 10 / 3 / 3; versioned if alternative designs later |

**Example:** Ratama's 15.5 ± 0.1 mm is represented as nominal 15.5, LSL 15.4, USL 15.6, unit mm; do not parse tolerance calculations from a free-text string alone.

### 3.3 Attribute study-only metadata — S05/S08

| Field key | UI label | Type | R/O/A | Rule |
| --- | --- | --- | --- | --- |
| `sample_count`, `appraiser_count`, `trial_count` | Study Design | Read-only preset | A | 50 / 3 / 2 |
| `categories` | Result Options | Read-only | A | Exactly OK, NG |
| `reference_source` | Reference Source / Standard | Text | O (recommended) | Provenance of known reference classification |
| `reference_verified_by` | Reference Verified By | Account/text | C | Defined when Quality policy requires validation |
| `judgment_policy_id`, `policy_version` | Active Judgment Policy | Approved selection | C | Only approved versions eligible for automatic disposition |
| `reference_status` | Reference Verification | Pending / Verified | A/C | Must be accepted under applicable policy before final automatic evaluation |
| `sample_id[1..50]` | Sample ID | Unique ID | R | Defaults 01–50; label can be customized |
| `reference[sample_id]` | Known Reference | OK/NG selector | R for Kappa | 50 independent reference labels; access-controlled and hidden during blind assessment |

**Critical:** Missing reference ≠ OK. Never generate/fill a known reference from an appraiser's majority vote.

### 3.4 Appraisers & Samples — S06

| Field | R/O/A | Rule |
| --- | --- | --- |
| Appraiser #1, #2, #3 — display name | R | Nonempty; unique within study; this person measured/inspected and may not be the logged-in account |
| Appraiser employee code | O | Useful identifier without requiring that each appraiser logs in |
| Part/sample ID 01..N | A/O | System sequence generated; human-friendly custom label optional |
| Input completion per appraiser | A | 0/30 Variable or 0/100 Attribute |
| Person entering values | A | Per-edit actor from current authenticated session |

**No requirement** that every appraiser have a software login in MVP. A single authorized owner/Supervisor can transcribe combined paper or three separate paper sheets into the *same* study.

## 4. Screen-specific actions & components

### S01 — Login

- **Fields:** email (R), password (R), show/hide password control.
- **Actions:** Sign In; Sign Out from authenticated shell.
- **Validations:** missing values, failed sign-in generic message, disabled account rejected; avoid leaking account existence; no public Sign Up.
- **Security:** server authorization, rate limit/session expiration and password hashing to be defined in architecture.

### S02 — Dashboard

- **Display:** total accessible studies, Draft, Finalized, Review Required, recent studies (name, type, date, status, result), Create New MSA.
- **Action:** open study or create new.
- **Scope:** operator sees own studies only by default until OPEN-005 is resolved; Supervisor sees all authorized studies. Admin gets User Management instead of automatic study access.
- **Constraint:** do not interpret *Draft/Finalized* (lifecycle) as *OK/NG* (statistical disposition).

### S03 — MSA History

- **Columns:** Study ID, Name, Type, Part/Characteristic, Study Date, Owner, Lifecycle Status, Quality Result, Updated At, Revision.
- **Search:** ID, name, part name/number or characteristic.
- **Filters:** Variable/Attribute, Draft/Finalized, quality result, date range, owner (Supervisor only).
- **Actions:** View Detail; Continue Draft (if authorized); Export PDF of Final; Revision/Soft Delete (Supervisor only).
- **Rules:** pagination; no hard delete; do not accidentally treat a soft-deleted study as ordinary active history.

### S04 — Create New MSA

- Two selectable methods: **Variable MSA** / **Attribute MSA** with short descriptions.
- **Action:** Create & Continue → S05. Allocates Study ID and Draft under owner.
- No unsupported ANOVA, nested, bias, linearity, stability, SPC, or open-ended plugin selection in MVP.

### S05 — Study Setup

- Common study metadata from §3.1.
- Type-specific metadata from §3.2 or §3.3.
- **Actions:** Back, Save Draft, Continue.
- **Rule:** preset statistics are read-only; optional values are never silently invented. User can continue workflow with incomplete metadata, but S09 Calculate validates required fields.

### S06 — Appraisers & Sample Setup

- Three appraiser name rows; 10 Variable part IDs or 50 Attribute sample IDs; compact completion summary.
- **Actions:** Back, Save Draft, Continue to S07/S08.
- **Validation:** no duplicate appraiser identities or part IDs; unique normalized IDs, no empty required names before Calculate.
- **Change handling:** changing an appraiser or sample ID after measurements exist must preserve correct mapping; replacing sample count/design is not allowed in preset MVP.

### S07 — Variable Measurement Input

**Layout:** three appraiser tabs, each a 10-part grid with 3 trial numeric columns:

| Part ID | Trial 1 | Trial 2 | Trial 3 |
| --- | --- | --- | --- |
| 01 | [number] | [number] | [number] |
| 02–10 | [number] | [number] | [number] |

- **Total:** 30 numeric values/tab × 3 = **90 readings**.
- **Actions:** Save Draft, Previous, Next Appraiser, Paste from Excel, Calculate when complete.
- **UX:** sticky row/column labels where feasible; progress 0/30 per tab and 0/90 total; visibly highlight invalid/missing cells.
- **Input parsing:** allow decimal separator formats only as explicitly supported; normalize before computation; preserve source precision; reject nonnumeric/NaN/Infinity, extra pasted columns, duplicate slot writes outside an explicit edit.
- **Calculated-only:** R, Rbar, EV, AV, GRR, PV, TV and UCL are NEVER editable fields.
- **Editing after Calculate:** mark result Stale, clear its validity status, require recalculation before Finalize.

### S08 — Attribute Assessment Input

**Layout:** three appraiser tabs, each 50 rows with Trial 1 and Trial 2 selectors:

| Sample ID | Trial 1 | Trial 2 |
| --- | --- | --- |
| 01 | OK / NG / blank | OK / NG / blank |
| 02–50 | OK / NG / blank | OK / NG / blank |

- **Total:** 100 appraiser decisions/tab × 3 = **300 decisions**.
- **Actions:** Save Draft, Previous, Next Appraiser, Paste OK/NG values from Excel.
- **Reference:** separate **Known Reference** subview with 50 OK/NG values, reference source and verification metadata; restricted per blind-study permission rule. Input completeness is distinct for decisions (0/300) and references (0/50).
- **Important flow:** source may be one combined physical form or three sheets; users can fill tabs on different occasions, preserving **one study ID**.
- **Categories:** exact OK/NG; user-friendly import normalization can accept only documented equivalents after explicit preview; no arbitrary free-text.
- **Calculate:** only if all required metadata, three appraiser identities, 300 decisions, 50 known references and applicable reference validation pass.
- **Safety:** appraiser assessment view must not display the reference during blind collection.

### S09 — Results & Review (shared layout; type-specific panel)

**Shared header:** study identity, version, calculated-at, method/version, input completeness, policy/version, calculation validity (Current/Stale), warnings and recommended action.

**Variable results (primary):**
- %GRR Study Variation with Ratama band (<10 OK, 10–30 Conditional/Special OK, >30 NG), NDC and criterion NDC≥5, R-chart range signals.
- %Tolerance if valid LSL/USL; clearly named as a *separate* fitness-for-purpose ratio.
- Details accordion: Rbar, EV, AV, GRR, PV, TV, Rp, XDIFF, per-subgroup range and method constants; charts only when supported/validated.
- **Disposition guard:** if assessment is Conditional, NDC is deficient, R-chart alarms exist, or a critical statistic is undefined, present **Review Required** independently from the %GRR band. %Tolerance's role in final acceptance depends on client policy.

**Attribute results (primary):**
- Three separate appraiser rows/cards: **Ratama Pooled Kappa vs Known Reference** and source-band OK/Improvement/Poor (subject to approval).
- For each appraiser: decision-level correct agreement, within-appraiser same-label consistency, percentage of samples correct on both trials, missed-NG and false-reject rates **with explicit numerator/denominator**.
- Details: 2×2 confusion counts A/B/C/D; Po, Pe and calculation method version.
- Combined study view: show all appraiser outcomes; **do not compute an unapproved mean Kappa or fabricate an overall pass**.
- Applied **approved** policy and version determine any automatic overall verdict; without such a policy, show **Review Required / No Approved Acceptance Policy**.

**Actions:** Recalculate (if input changed), Back to Input, Review, Finalize (authorization & guards), PDF Preview. Results can be viewed in Draft, but final PDF is only a finalized, immutable report.

### S10 — Study Detail & Revisions

**Tabs/sections:** Overview, Input Data (read-only if Final), Results, Report, Change History, Revisions.
- **Display:** who created, who entered/changed which reading, source appraiser, timestamps, method version, policy snapshot, status and finalization details.
- **Actions:** Continue Draft (owner/Supervisor); Create Revision (Supervisor); Soft Delete (Supervisor); Open historical finalized versions.
- **Rules:** finalized data never edited in place; revision begins a new editable draft linked to parent; previous final result/PDF remains downloadable and unchanged.
- **No hard delete** from normal UI.

### S11 — Report Preview / PDF

**Report header:** company/study identity, study date, part/characteristic, gage or inspection method, appraiser names, performed by, revision, Finalized At, method/calculation version and judgment-policy version.

**Variable report:** setup & tolerance, input summary, EV/AV/GRR/PV/TV, %StudyVar, %Tolerance if valid, NDC, R-chart warnings, source judgment and Supervisor disposition/reason if applicable.

**Attribute report:** 50-reference composition, appraiser-level 2×2 confusion values, pooled Kappa, agreement/repeatability, missed-NG/false-reject rates and denominators, policy/review status and notes.

**Actions:** Preview, Download PDF, Print; previous finalized revision PDF should be reproduced from its immutable stored snapshot/artifact, not re-rendered using today's changed policy.
- Draft preview (if offered) must be visibly **DRAFT**, non-certifying, and may be omitted from MVP.
- Report must not claim AIAG/Minitab certification or formal Quality acceptance without evidence.

### S12 — User Management (Admin)

| Field / component | Rule |
| --- | --- |
| Name | R; human name |
| Email | R; unique, normalized |
| Assigned role | R: Admin / Supervisor / Operator |
| Account status | Active / Inactive |
| Initial password / reset | Secure temporary activation/reset mechanism; no plaintext password display |
| Last login / created at | Audit/read-only where available |

**Actions:** Create Account, Update Role, Activate/Deactivate, Reset Access.
- **No self-registration**.
- Admin **does not automatically obtain Supervisor's statistical approval powers**.

### S13 — Judgment Policy (Supervisor)

| Field | R/O/A | Rule |
| --- | --- | --- |
| Policy Name, ID, Version | R/A | Named, versioned; previous approved versions immutable |
| Policy Scope / Characteristic applicability | R/C | Defined business scope; no multi-tenant module |
| Kappa bands | R/A | Ratama preset thresholds initially; approved override policy only |
| Missed NG maximum | C | Must come from Quality/customer approval, never guessed as universal |
| False Reject maximum | C | Ditto |
| Overall study disposition rule | C | Explicit approved rule across appraisers; no automatic mean Kappa |
| Approval authority & evidence reference | R to activate | No requirement for fourth app role; approval evidence recorded |
| Effective date, version reason | R | Version history & audit |
| Policy status | A | Draft / Approved / Active / Superseded (subject to workflow detail) |

**Actions:** View Preset, Create Version Draft, Record Approval, Activate Approved Version, Compare Versions.
- Only Supervisor can configure/select; Operator cannot change rules.
- A Supervisor's ability to enter criteria **does not itself imply Quality approval**. Activation requires recorded approval evidence, and actor/time are audited.
- Formula definitions never exposed for editing. Per-final-study policy version/content is snapshotted.
- Numeric missed-NG/false-reject thresholds and precise overall aggregation **remain domain OPEN**.

## 5. Authorization matrix — MVP recommendation

| Feature | Admin | Supervisor | Operator |
| --- | --- | --- | --- |
| Login / self session | Yes | Yes | Yes |
| Manage accounts / roles | Yes | No | No |
| View dashboard/history | No study access by default | All authorized studies | **Own studies** by safe default |
| Create study / own draft | Not by Admin role alone | Yes | Yes |
| Edit nonfinalized study | No by Admin role alone | Any authorized study | Own only |
| Enter Attribute reference | No by Admin role alone | **Recommended** | **OPEN** (requires quality/blinding rule) |
| Run calculation / view results | No by Admin role alone | Yes | Own studies |
| Edit/activate judgment policy | No | Approved rules only | No |
| Finalize normal own study | No | Yes | **OPEN** (recommend only if policy valid, no supervisor-review flags) |
| Finalize with Review Required | No | **Supervisor disposition, subject to Quality policy** | No |
| Create revision of Final | No | Yes | No |
| Soft delete | No | Yes | No |
| View audit events | Accounts only | Studies within scope | Own-study events only |
| Download final PDF | No by Admin role alone | Yes | Own studies |

**Important:** Operator visibility outside own studies, reference entry authority, and exact Finalize approval are deliberate OPEN decisions. Apply the more restrictive option until formally approved. Authorization must be enforced server-side, not just by hiding buttons.

## 6. Validation and state rules

### 6.1 Completeness

| Dataset | Draft save | Calculate | Finalize |
| --- | --- | --- | --- |
| Common setup | Partial allowed | Required fields valid | Must remain valid |
| Variable | 0..90 numeric readings | Exactly 90 valid | Current calculation + permitted decision |
| Attribute decisions | 0..300 decisions | Exactly 300 valid OK/NG | Current calculation + permitted decision |
| Attribute known reference | 0..50 | Exactly 50 valid + required verification | Appropriate policy/review |
| Approved policy | Not required for collecting data | Mathematical metric calculation may remain available without policy; **automatic acceptance judgement forbidden** | Policy-approved disposition or explicit documented Supervisor review required |

**Clarification:** If data are mathematically complete but quality policy is not approved, **Calculate statistical metrics** may run (so work isn't blocked), but quality **pass/fail must remain Review Required**. An unapproved policy must not suppress metrics.

### 6.2 Transition model

```text
Draft --Save Draft--> Draft
Draft --Calculate (valid inputs)--> Draft + Current Result
Draft + changed input --> Draft + Stale Result (must recalculate)
Draft + Current Result + authorization/policy checks --Finalize--> Finalized vN
Finalized vN --Supervisor Create Revision--> Draft Revision vN+1
Draft Revision vN+1 --Finalize--> Finalized vN+1 (vN retained)
Finalized vN --Soft Delete--> Hidden from active history, retained for audit
```

No field should allow ordinary edits to a Finalized snapshot. State transitions must be transactionally verified by backend.

### 6.3 Error / edge states

- Inline missing/invalid cell messages with row/appraiser/trial identification; show summary of issues.
- No silent coercion of blanks to zero; no negative AV, NaN, Infinity or default Kappa=1 when Pe=1.
- Numeric measurement may be zero/negative when scientifically valid; do not prohibit it simply because AV (a variation metric) cannot be negative.
- Tolerance needs coherent units, LSL and USL and USL>LSL; if absent, %Tolerance is **Not Calculable**, not 0 and not a blocker for %StudyVar.
- If GRR=0, NDC Not Calculable; if TV=0, %StudyVar Not Calculable.
- If a reference class is absent, corresponding class-conditional error rate Not Calculable; no invented 0%.
- Enforce inputs by exact preset positions; safely handle pasted columns, trailing whitespace, size mismatch and duplicate IDs.
- Confirm unsaved changes on navigation. Saving and recalculation failures must never erase valid draft data.
- All threshold comparisons on unrounded values; preserve original measurement input representation/precision.
- Policy changes don't silently retroactively change results of old finalized studies.

## 7. Reusable UI components

- App shell: navigation, user menu, sign-out, study context breadcrumbs.
- Wizard stepper: Setup → Appraisers & Samples → Measurements/Assessments → Results.
- Editable grid: keyboard navigation, cell validation, paste preview and progress counter.
- Status indicators that distinguish **lifecycle status** from **calculation/policy judgment**.
- Read-only statistic panel + optional detailed calculation section.
- Supervisor review/exception area (reason + actor + timestamp).
- Version/history viewer and printable report.

**Responsiveness:** Desktop-first for large data grids; mobile may show read-only overview and horizontal-scrolling/editing fallback, with no claim of a fully optimized 90/300-cell mobile editor until tested. Accessibility: visible input labels, keyboard navigation, non-color-only status cues and explicit error text.

## 8. Acceptance checks for screen specification

| ID | Scenario | Expected |
| --- | --- | --- |
| UI-01 | Operator creates Variable and saves after only five cells | Same Draft and five values persist |
| UI-02 | Paste valid 10×3 Variable tab values | Exactly 30 slots filled; remaining tabs unchanged |
| UI-03 | Missing one of 90 Variable measurements | Calculate blocked, pinpoint missing cell |
| UI-04 | Attribute outcomes received on 3 different paper forms | One study, three appraiser tabs, one total 300 decision dataset |
| UI-05 | Attribute decisions complete, references missing | No Kappa vs reference; Draft preserved |
| UI-06 | Attribute assessments being collected blind | Reference values not revealed to appraiser/unauthorized account |
| UI-07 | Attribute Kappa calculated under unapproved acceptance policy | Metrics visible, quality verdict Review Required/No Approved Acceptance Policy |
| UI-08 | Operator tries editing another user's draft | Backend denies access, not merely hidden UI |
| UI-09 | Result calculated, then one input changed | Result marked Stale; Finalize blocked until recalculated |
| UI-10 | Supervisor creates revision of finalized study | Old version and PDF unchanged, new revision editable |
| UI-11 | Supervisor activates unapproved policy | Denied; requires recorded Quality approval evidence |
| UI-12 | Policy thresholds modified after old study finalized | Old PDF and judgement unchanged; new version only applies prospectively |
| UI-13 | Appraiser shows 100% repeated-label agreement but high missed-NG | Separate figures, no false “100% correct” description |
| UI-14 | Nonnumeric value pasted into Variable table | Inline error; no NaN/Infinity displayed |
| UI-15 | Admin account without Supervisor role opens policy/study edit | Access denied by server |

## 9. MVP exclusions

No SPC, multi-client tenant management, automatic full-file Excel import, ANOVA/nested GRR, custom statistical formula editor, multi-level approval hierarchy, mandatory appraiser logins, parallel section-level multi-user editing, mobile-native app, broad notifications or Minitab output-clone UI.

## 10. Explicit open questions for final screen review (do not block DRAFT)

1. **Attribute reference entry:** Supervisor-only by default vs authorized Operator transcription after blind inspection (requires proper control).
2. **Finalize:** May Operator finalize clean own studies, or all studies require Supervisor review? How is exception disposition recorded?
3. **Historical visibility:** Should Operators see only own studies (recommended secure MVP default) or all completed reports?
4. **PDF layout & branding:** company name/logo, signature/quality approval evidence and whether full data appendix is required.
5. **Policy approval:** exact Quality approver/evidence and industry/customer-specific limits for missed-NG, false reject and overall judgement.
6. **Study identification:** how the client's internal study ID conventions and date/timezone should be displayed.

**Document state:** Product design candidate only. Requires stakeholder review, QA, and domain approval; **NOT FROZEN**, no application code written.
