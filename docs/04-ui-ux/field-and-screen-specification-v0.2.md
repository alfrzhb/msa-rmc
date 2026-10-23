# RMC MSA — Field & Screen Specification v0.2

**Status:** REVISED DRAFT — FOR REVIEW, NOT FROZEN  
**Tanggal:** 23 Oktober 2026  
**Produk:** RMC MSA Analysis Tool (MVP, satu perusahaan klien)  
**Basis:** Requirements Analysis v0.3; Ratama Variable/Attribute XLSX; Ratama PPT; historical Field & Screen v1.0 (reference only); Calculation Decisions v0.3; Attribute Study Input Workflow v0.1; Attribute Judgment Policy v0.1.

> Tujuan: menetapkan screen, field, aksi, validasi, dan hak akses untuk MVP sebelum membuat wireframe/implementasi. Isi form diturunkan dari *makna data* Ratama, bukan dari koordinat sel Excel. Keputusan yang belum disetujui sebagai kebijakan Quality harus dinyatakan OPEN, bukan diasumsikan FROZEN.

**Perubahan dari dokumen historis:** Field & Screen v1.0 lama tidak memuat role/login dan pengaturan judgment policy. Dokumen rebuild ini mengikuti keputusan terkini tentang Admin/Supervisor/Operator, study audit, dan kebijakan Attribute—tanpa menganggap freeze historis sebagai persetujuan baru.

## 0. Perubahan sejak v0.1

| Area | v0.2 |
| --- | --- |
| Study Setup | Field wajib ditampilkan pertama; informasi tambahan pada **Advanced Details** yang tertutup secara default |
| Finalize | Operator menghitung/menyimpan dan mengajukan review; **Supervisor-only Finalize sebagai default konservatif yang masih perlu ratifikasi Ratama**, tanpa multi-level approval |
| Attribute input | Tetap **1 study + 3 appraiser tabs**, Known Reference menjadi subview terpisah, dibatasi aksesnya dan tidak bocor selama pemeriksaan buta |
| Results | **Judgement → warnings → key metrics** di atas; rumus/konstanta/confusion detail dalam expandable **Calculation Details** |
| Konsistensi | Permission, state transition, dashboard/history, laporan, field legend, dan QA scenarios diperbarui mengikuti alur di atas |

v0.1 dipertahankan sebagai riwayat. Dokumen referensi Ratama yang lama berstatus FROZEN tidak menjadikan spesifikasi rebuild ini FROZEN.

## 1. Prinsip desain

1. **Sederhana:** Login → Dashboard/History → Create Study → Setup singkat → Input → Calculate → Submit for Review → Supervisor Finalize → PDF.
2. **Satu studi, satu sumber data:** Satu studi mencakup seluruh appraiser dan referensi; bukan satu studi per kertas atau appraiser.
3. **Preset awal:** Variable 10 parts × 3 appraisers × 3 trials; Attribute 50 samples × 3 appraisers × 2 trials. Ukuran preset hanya dapat diganti pada versi metode/desain yang telah divalidasi.
4. **Excel-friendly input:** tab per appraiser, keyboard navigation, paste rectangular dari Excel, validasi cell-level; tidak ada XLSX file import dalam MVP.
5. **Progressive disclosure:** tampilkan hanya metadata wajib pada Setup, sisanya dalam **Advanced Details**. Save Draft kapan saja; Calculate memerlukan data valid, tetapi navigasi dan penyimpanan Draft tidak terkunci.
6. **Peran & kepemilikan:** Operator membuat/mengedit studi miliknya, Supervisor mengelola studi dan revisi, Admin mengelola akun. Hak akses pada server wajib sama dengan yang ditampilkan UI.
7. **Audit & reproduktibilitas:** studinya menyimpan identitas appraiser (orang yang mengukur) terpisah dari akun penginput; versi formula dan policy, snapshot hasil final, audit log, serta histori revisi.
8. **Perhitungan bukan formulir bebas:** EV/AV/GRR dan Kappa dihitung otomatis, bukan field yang dapat diketik. Formula tidak bisa diedit pengguna.
9. **Referensi Attribute:** Known Reference OK/NG berada di subview terpisah, hanya role berwenang (default **Supervisor**) boleh melihat/menginputnya; appraiser dan layar koleksi buta tidak pernah menampilkan referensi sebelum klasifikasi selesai. Permission final tetap memerlukan approval Ratama.
10. **Results first:** judgement dan warning harus tampak tanpa scrolling ke rincian formula, sedangkan formula/konstanta disembunyikan pada expandable details.
11. **Kebijakan keputusan:** Standard Preset + Configurable **Approved** Policy. Hanya Supervisor yang memilih/mengelola kebijakan yang sudah memiliki persetujuan Quality. Tanpa kriteria wajib yang sah, laporan menampilkan perhitungan dan **Review Required**, bukan status kelulusan otomatis.

## 2. Screen inventory dan navigasi

| ID | Screen | Tujuan utama | Peran |
| --- | --- | --- | --- |
| S01 | Login | Autentikasi individual | Public (login) |
| S02 | Dashboard | Ringkasan studi dan pintasan | Operator, Supervisor (sesuai scope) |
| S03 | MSA History | Cari/filter/buka studi | Operator, Supervisor (sesuai scope) |
| S04 | New MSA — Type Selection | Pilih Variable / Attribute | Operator, Supervisor |
| S05 | Study Setup | Metadata wajib + Advanced Details (collapsed) | Pemilik studi/Supervisor |
| S06 | Appraisers & Samples | Isi 3 appraiser + ID part/sampel | Pemilik studi/Supervisor |
| S07 | Variable Measurement Input | 90 angka, tiga tab appraiser | Pemilik studi/Supervisor |
| S08 | Attribute Assessment Input | 300 OK/NG pada 3 tab; restricted Known Reference subview | Pemilik studi/Supervisor; reference: kontrol tambahan |
| S09 | Calculation Results & Review | Judgement + warning utama, calculation details expandable, Submit/Finalize | Pemilik studi/Supervisor, tergantung policy |
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
       │                        ├─ Submit for Review (Operator)
       │                        └─ Supervisor Finalize
       └─ S10 Study Detail ◀────────┘
            ├─ S11 Report / PDF
            └─ Revision history
S12 User Management (Admin)
S13 Judgment Policy (Supervisor)
```

**Reuse:** S05/S06/S09/S10/S11 are shared components/flows with type-specific fields and result panels. Four visible steps: **Setup → Appraisers & Samples → Data Input → Results & Review**; S05/S06 may use separate panels inside the wizard. UI may implement S05–S09 as a wizard with progress steps; they do not have to be five unrelated pages.

## 3. Data dictionary dan ownership

### 3.1 Common Study metadata — S05

**Legend:** R = required before **Calculate**; O = optional; A = automatic/read-only; C = conditional. Draft may be saved with incomplete R fields. **Visibility is separate from requiredness**: Core/Advanced/System; field definitions below remain authoritative.

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

**S05 visibility contract (applies to §3.1–3.3):** `study_name`, `study_date`, `part_name`, and `characteristic` are **Core**. `msa_type` and generated ID are compact read-only header values. `part_number`, `model`, `process_line`, `study_context`, `performed_by`, `notes` are **Advanced Details**. Owner, status, timestamps and revision are System/read-only on demand. **Advanced does not mean optional**: should future Quality policy require a field, validation still applies.

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

**Variable Core:** `gage_name`, `unit`, plus a simple **Specification available?** toggle. If Yes, ask both `lsl` and `usl`, with clear numeric validation; nominal target may remain Advanced. If No, do not invent values; show %Tolerance Not Calculable. **Variable Advanced Details:** `gage_number`, `gage_type`, `nominal_value`, `gage_resolution`, `calibration_reference`, and `specification_note`. Study design 10×3×3 is shown as a noneditable preset badge.

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

**Attribute Core:** the four common Core fields plus 50×3×2 preset and OK/NG legend. **Attribute Advanced Details:** `reference_source`, `reference_verified_by`, and policy information appear in the restricted reference/review context, never embedded in the operator's assessment grid; do not hide approval evidence when required.

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

- **Primary:** Create New MSA, Continue Draft, **Awaiting Supervisor Review** count (Supervisor), recent studies.
- **Secondary:** accessible-study counts Draft, Finalized, Review Required and link to History; keep lifecycle and quality status separate.
- **Role scope:** Operator sees own studies by conservative default; Supervisor sees all authorized studies; Admin manages users and has no automatic study privileges.
- **Review indicator:** "Awaiting Supervisor Review" is a queue/workflow flag on Draft/Current Result, **not** a statistical pass band or an extra approval tier.
- **Empty state:** explain how to create the first study.

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

**Layout, above the fold (Core):**
- Shared: **Study Name, Study Date, Part/Product Name, Characteristic**.
- Variable: **Gage Name, Unit**, plus **Specification available?** toggle; when enabled, request numeric LSL and USL together.
- Attribute: short confirmation **50 samples, 3 appraisers, 2 trials, OK/NG**; Known Reference is **not** entered or shown here during blind collection.
- Study ID, MSA Type, role/ownership are visible as contextual **read-only** information, not extra mandatory text boxes.

**Expandable `Advanced Details` (collapsed by default):**
- Shared: Part No., Model, Process/Line, Performed By, Measurement/Inspection Method, Notes.
- Variable: Gage No., Gage Type, Nominal, Instrument Resolution, Calibration Reference.
- Attribute: reference provenance/verification is completed in its **restricted subview**, not in an appraiser's regular Setup panel.
- Optional fields are never filled with guessed values.

**Actions:** Back, Save Draft (even incomplete), Continue (without losing data).
**Validation:** required Core fields are checked before Calculate; if invalid, provide field-level messages and route back to Setup. Advanced fields still validated if supplied. No entire form of 20+ fields is shown by default.

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

**One study / three appraiser tabs.** Each tab shows 50 sample IDs and Trial 1 / Trial 2 columns. The 50 samples are **shared** across all tabs; a source consisting of one combined paper or three separate appraiser sheets is transcribed under the **same study ID**.

| Sample ID | Trial 1 | Trial 2 |
| --- | --- | --- |
| 01 | OK / NG / blank | OK / NG / blank |
| 02–50 | OK / NG / blank | OK / NG / blank |

**Primary tab view:**
- Appraiser A / B / C tabs, **0/100 per tab** and **0/300** total completion.
- Save Draft, Previous/Next Appraiser, Paste from Excel and inline validation.
- No Known Reference column, tooltip, color cue, statistics or derived correctness indicator while blind judgments are being collected.
- Data-entry actor and appraiser identity are recorded separately; Operator owner can transcribe outcomes from other appraisers' paper without gaining access to their user accounts.

**Restricted "Known Reference" subview (not a fourth appraiser):**
- 50 shared sample IDs, reference OK/NG dropdown, provenance and verification fields, **0/50** progress.
- **Default permission proposal: Supervisor-only** view/edit/verify; Operator and active appraiser assessment views cannot access the data, including via API or paste-error messages.
- The reference may be recorded by an authorized Supervisor separately, but must remain **masked until all independent appraiser trials are completed/locked**. The review of reference must not leak feedback that could influence unfinished trial data.
- If corrections are required after reference comparison, log the reason and revision; an unapproved operator must not use the reference as a guide to adjust answers.
- Confirm with Quality who may supply and approve each known reference; no assumed majority-vote or autofill.

**Calculation prerequisites:** valid common metadata, distinct appraisers, 300 OK/NG decisions, 50 **verified** known-reference labels. If incomplete, retain Draft and pinpoint missing cells. If policy thresholds are unapproved, statistical outputs can still be calculated but **overall pass must remain Review Required**.
**Never** invent missing categories or reveal reference answers to an appraiser during a blinded inspection.

### S09 — Calculation Results & Review (shared layout)

**Information hierarchy (MVP contract):**

1. **Judgement banner:** separate **Calculated Band** (method category) from **Overall Disposition** (approved policy outcome or Review Required). Display policy name/version and whether the decision is approved; no automatic OK without an approved rule.
2. **Warnings and next action:** NDC failed, R-chart alarm, Conditional band, missed NG, incomplete/invalid reference, no approved policy, or Stale Result. One clear sentence for each warning, with jump-to-fix/review.
3. **Key statistics, at a glance:** Variable %GRR Study Var, NDC, %Tolerance when defined; Attribute one compact row per appraiser showing pooled Kappa and its band, missed NG and false reject. Show percentages and denominators where scientifically necessary.
4. **Expandable `Calculation Details` (collapsed by default):** reproducible formulas, method and constants/version, Variable EV/AV/GRR/PV/TV, Rbar/Rp/XDIFF, subgroup ranges and graph data; Attribute Po/Pe, confusion A/B/C/D per appraiser, agreement and within-appraiser/both-trials metrics.
5. **Review & actions** at the end: interpretation, evidence and reason, Calculate/Recalculate, Back to Input, Save Draft, **Submit for Review** (Operator/owner), **Finalize** (Supervisor only as conservative proposal), PDF access after Finalized.

**Variable:**
- Calculated band: <10% OK; 10–30% Special OK/Conditional; >30% NG, using unrounded %Study Var.
- NDC≥5 and R-chart status independently visible; any conflict, Conditional band or undefined result becomes **Review Required** as separate workflow indicator.
- %Tolerance is displayed separately where both specification limits are valid. No fabricated %Tolerance acceptance threshold.
- No analysis of ANOVA or destructive GRR without a separate validated method.

**Attribute:**
- Per-appraiser pooled Kappa vs known reference, OK/Improvement/Poor band; Missed NG = C/(A+C) and False Reject = B/(B+D), each with denominator. Do not call repeatability "accuracy."
- One study overview **never** silently averages three Kappa values into an unapproved global pass.
- Only the versioned, **approved** policy may yield an overall automatic disposition. An unapproved or incomplete risk policy yields **Review Required / No Approved Acceptance Policy**, with measured metrics intact.
- Do not show a Known Reference comparison/derived correctness to an appraiser while any blinded trial is still in progress.

**Finalize recommendation (not yet ratified by Ratama):**
- Operator can **Calculate, Save Draft, Submit for Review**; **cannot directly Finalize**. A submitted study stays logically a Draft with review flag, so this is not a new approval hierarchy.
- Supervisor checks full input, whether the result is Current, reference verification and approved policy. Clean studies may be **Finalized** by Supervisor under validated rules.
- Exceptions or missing approval **must not be relabeled OK**. Supervisor can record **Review Required / Not Approved** with reason for an audit-only final report *if the client authorizes that disposition*. Until authorized, keep in review queue; no automatic release.
- Finalize requires explicit confirmation and records actor/time, method version, policy snapshot, disposition and reason. Old finalized versions stay immutable.

### S10 — Study Detail & Revisions

**Tabs/sections:** Overview, Input Data (read-only for Final), Results, Reports, Change History, Revisions.
- **Review queue:** for submitted Draft studies, show Submitted By/At, calculated result freshness and Supervisor action. Submission is **not** Finalized.
- **Display:** who created, who entered/changed each reading, appraiser identity, timestamps, formula version, approved policy snapshot, calculated band, overall disposition, Supervisor reviewer identity and review reason.
- **Actions:** Continue Draft (owner/Supervisor); Withdraw/Resubmit Draft review request (proposed, logged); Create Revision of Final (Supervisor); Soft Delete (Supervisor); Open old versions.
- **Rules:** finalized source data and PDF immutable; revision creates a new editable version linked to parent; prior Final and policy remain reproducible.

### S11 — Report Preview / PDF

**First page hierarchy:** study identity, date, method/version, **Overall Disposition**, primary statistics and risk warnings; show policy ID/version and review evidence. Follow with appraiser/measurement context, summary and detailed appendix if required.

**Variable:** %Study Var band, NDC status, %Tolerance when valid, R-chart warning and supporting EV/AV/GRR/PV/TV in detail.
**Attribute:** **separate appraiser Kappa/band**, missed-NG/false-reject counts and rates, repeatability, reference composition, and explicit policy/review status; no invented overall pooled Kappa.
**Actions:** Preview, Download PDF, Print final. Only finalized PDF is immutable. Previous revision PDF must not be regenerated under changed policy.
**Restriction:** draft preview (if supported) carries **DRAFT — NOT APPROVED** watermark/text; report cannot claim customer/AIAG/Minitab certification without evidence.
**Open:** corporate branding/logo, signatures and full sample-reading appendix need Ratama review.

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

## 5. Authorization matrix — MVP conservative proposal (Ratama approval pending)

| Feature | Admin | Supervisor | Operator |
| --- | --- | --- | --- |
| Login / own session | Yes | Yes | Yes |
| Manage accounts / roles | Yes | No | No |
| View Dashboard / History | No statistical-study access by Admin role alone | Authorized all | Own studies by safe default |
| Create or edit Draft study | No by Admin role alone | Yes, including others | Own only |
| Enter Variable/Attribute appraiser decisions | No by Admin role alone | Yes | Own study (data transcription), without Known Reference access |
| View/edit/verify Known Reference | No by Admin role alone | **Recommended Yes** | **No by conservative default** |
| Calculate and read result | No by Admin role alone | Yes | Own study, but blind reference results must not leak before inspection ends |
| Save Draft / Submit for Review | No by Admin role alone | Yes | Own study |
| Finalize & record disposition | No by Admin role alone | **Recommended Yes, with approved policy** | **No by conservative default** |
| Activate approved judgment policy | No | Yes, only after documented Quality approval | No |
| Create revision / soft delete | No | Yes | No |
| Audit trail and finalized PDF | Account scope only | Authorized study scope | Own-study scope |

**Default resolution:** Grant fewer rights until Ratama explicitly signs off; don't hide a button while leaving its backend endpoint accessible. The policy approval authority is a documented Quality person/process, not an extra MVP app role. Supervisor ownership of finalization is a **recommended design**, not a business rule marked FROZEN.

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
Draft --Calculate (complete valid inputs)--> Draft + Current Result
Draft + changed inputs/policy choice --> Draft + Stale Result
Draft + Current Result --Submit for Review--> Draft + Review Requested
Draft + Review Requested --Supervisor accepts clean result under approved policy--> Finalized vN
Draft + Review Requested --Supervisor requests correction--> Draft + Needs Correction
Draft + Review Requested --No approved policy / risk unresolved--> Review Required; cannot silently Finalize as OK
Finalized vN --Supervisor Create Revision--> Draft Revision vN+1
Finalized vN --Soft Delete--> Hidden in active history, retained for audit
```

"Review Requested" and "Needs Correction" are **review flags on a Draft**, not new statistical bands or a full multi-level approval workflow. A current calculation and permission check are always required for Finalize. Supervisor decision/reason/actor/time are recorded. Any final disposition of unresolved cases needs explicit Ratama policy authorization. No finalized snapshot is edited in place.

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
- Supervisor review/exception area (reason + actor + timestamp), with Submitted By/At and a correctable Draft flag.
- **Core + expandable Advanced Details** in Setup and **judgement-first + expandable Calculation Details** in Results.
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
| UI-16 | User opens Study Setup | Only Core fields visible; Advanced Details closed by default; optional values persist when toggled |
| UI-17 | Operator calculates a clean own study | Operator can Save/Submit, not Finalize in conservative MVP; supervisor queue receives it |
| UI-18 | Supervisor finalizes with correct current results and an approved policy | Finalized snapshot records reviewer/time, formula/policy version and disposition |
| UI-19 | Supervisor sees no approved policy or unacceptable/undefined risk | No automatic OK; Review Required with documented exception handling |
| UI-20 | Operator is actively collecting Attribute blind trials | Neither UI nor API exposes Known Reference or correctness-derived signals |
| UI-21 | Three paper sheets arrive on separate dates | All decision values preserved in one study with independent appraiser tab progress |
| UI-22 | Results first loaded | Judgement and warnings visible immediately; formula and constants hidden until user expands Calculation Details |
| UI-23 | User returns from Results to change one input | Previously calculated result marked Stale, review submission invalidated until recalculated |
| UI-24 | Supervisor enters 50 verified reference values | Reference is not an appraiser tab; reference labels stay masked from unauthorized users |
| UI-25 | Policy gets updated after old study Finalized | Historical judgement and report stay exactly as frozen in prior study version |

## 9. MVP exclusions

No SPC, multi-client tenant management, automatic full-file Excel import, ANOVA/nested GRR, custom statistical formula editor, multi-level approval hierarchy, mandatory appraiser logins, parallel section-level multi-user editing, mobile-native app, broad notifications or Minitab output-clone UI.

## 10. Explicit open questions for final screen review (do not block DRAFT)

1. **Attribute reference entry:** Supervisor-only is the **conservative v0.2 default**, pending Ratama validation of authorized Quality reference custodians and the blind/unblind sequence.
2. **Finalize:** Conservative MVP **proposes Supervisor-only Finalize** (Operator submits), pending Ratama confirmation; define whether an unresolved finding may be finalized *for audit only* with an explicit Not Approved disposition.
3. **Historical visibility:** Should Operators see only own studies (recommended secure MVP default) or all completed reports?
4. **PDF layout & branding:** company name/logo, signature/quality approval evidence and whether full data appendix is required.
5. **Policy approval:** exact Quality approver/evidence and industry/customer-specific limits for missed-NG, false reject and overall judgement.
6. **Study identification:** how the client's internal study ID conventions and date/timezone should be displayed.

**Change control:** v0.1 remains in GitHub history and as a standalone file. No historical file is overwritten by v0.2.

**Document state:** Product design candidate only. Requires stakeholder review, QA, and domain approval; **NOT FROZEN**, no application code written.
