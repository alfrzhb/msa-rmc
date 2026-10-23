# RMC MSA — Field & Screen Specification v0.3

**Status:** PRODUCT-OWNER ROLE/APPROVAL DECISIONS ACCEPTED — DRAFT, NOT FROZEN  
**Tanggal:** 23 Oktober 2026  
**Produk:** RMC MSA Analysis Tool (MVP, satu perusahaan klien)  
**Basis:** Requirements Analysis v0.3; Ratama Variable/Attribute XLSX; Ratama PPT; historical Field & Screen v1.0 (reference only); Calculation Decisions v0.3; Attribute Study Input Workflow v0.1; Attribute Judgment Policy v0.1.

> Tujuan: menetapkan screen, field, aksi, validasi, dan hak akses untuk MVP sebelum membuat wireframe/implementasi. Isi form diturunkan dari *makna data* Ratama, bukan dari koordinat sel Excel. Keputusan yang belum disetujui sebagai kebijakan Quality harus dinyatakan OPEN, bukan diasumsikan FROZEN.

**Perubahan dari dokumen historis:** Field & Screen v1.0 lama tidak memuat role/login dan pengaturan judgment policy. Dokumen rebuild ini mengikuti keputusan terkini tentang Admin/Supervisor/Operator, study audit, dan kebijakan Attribute—tanpa menganggap freeze historis sebagai persetujuan baru.

## 0. Perubahan sejak v0.2 — keputusan 23 Oktober 2026

Empat jawaban product-owner diterima sebagai **arah fungsi MVP**, bukan pembekuan rumus/acceptance threshold atau pengesahan Quality Ratama.

| Pertanyaan v0.2 | Keputusan yang disetujui | Dampak |
| --- | --- | --- |
| Finalize | **Supervisor saja** | Operator Calculate → Save/Submit for Approval; Supervisor Approve & Finalize |
| Attribute Known Reference | **Hanya Supervisor mengelola/mengubah/memverifikasi** | Restricted view/write; tidak muncul pada form blind appraiser |
| History | **Operator dapat melihat histori seluruh studi** | Read-only untuk studi orang lain, edit Draft sendiri; referensi blind tetap dilindungi |
| PDF | **Memiliki logo dan tanda tangan persetujuan** | Nama/jabatan approver, tanggal approval, signature asset, policy snapshot pada laporan Final |

**Approval is not merely a signature image on the PDF.** Event persetujuan terjadi melalui aksi **Approve & Finalize** oleh Supervisor. Logo/tanda tangan adalah representasi laporan dari approval tercatat, bukan pengganti otorisasi server. Approval hanya **satu tingkat**; tidak ada workflow Head/Manager.

Logo dan tanda tangan aktual belum tersedia. Sistem menggunakan field dan slot aset sampai Supervisor/Quality memberikan aset berwenang. Implementasi signature image **bukan** certified cryptographic digital signature; jangan mengklaim keabsahan e-signature tersertifikasi tanpa mekanisme/legal review terpisah.

### Riwayat dari v0.1 ke v0.2

| Area | v0.2 |
| --- | --- |
| Study Setup | Field wajib ditampilkan pertama; informasi tambahan pada **Advanced Details** yang tertutup secara default |
| Finalize | Operator menghitung/menyimpan dan mengajukan review; **Supervisor-only Finalize sebagai default konservatif yang masih perlu ratifikasi Ratama**, tanpa multi-level approval |
| Attribute input | Tetap **1 study + 3 appraiser tabs**, Known Reference menjadi subview terpisah, dibatasi aksesnya dan tidak bocor selama pemeriksaan buta |
| Results | **Judgement → warnings → key metrics** di atas; rumus/konstanta/confusion detail dalam expandable **Calculation Details** |
| Konsistensi | Permission, state transition, dashboard/history, laporan, field legend, dan QA scenarios diperbarui mengikuti alur di atas |

v0.1 dipertahankan sebagai riwayat. Dokumen referensi Ratama yang lama berstatus FROZEN tidak menjadikan spesifikasi rebuild ini FROZEN.

## 1. Prinsip desain

1. **Sederhana:** Login → Dashboard/History → Create Study → Setup singkat → Input → Calculate → Submit for Approval → Supervisor Approve & Finalize → PDF.
2. **Satu studi, satu sumber data:** Satu studi mencakup seluruh appraiser dan referensi; bukan satu studi per kertas atau appraiser.
3. **Preset awal:** Variable 10 parts × 3 appraisers × 3 trials; Attribute 50 samples × 3 appraisers × 2 trials. Ukuran preset hanya dapat diganti pada versi metode/desain yang telah divalidasi.
4. **Excel-friendly input:** tab per appraiser, keyboard navigation, paste rectangular dari Excel, validasi cell-level; tidak ada XLSX file import dalam MVP.
5. **Progressive disclosure:** tampilkan hanya metadata wajib pada Setup, sisanya dalam **Advanced Details**. Save Draft kapan saja; Calculate memerlukan data valid, tetapi navigasi dan penyimpanan Draft tidak terkunci.
6. **Peran & kepemilikan (agreed):** Operator membuat/mengedit Draft miliknya sendiri dan **melihat histori semua studi** secara read-only di luar miliknya; Supervisor mengelola, **Approve & Finalize**, revisi dan Known Reference; Admin mengelola akun. Hak akses pada server wajib sama dengan UI.
7. **Audit & reproduktibilitas:** studinya menyimpan identitas appraiser (orang yang mengukur) terpisah dari akun penginput; versi formula dan policy, snapshot hasil final, audit log, serta histori revisi.
8. **Perhitungan bukan formulir bebas:** EV/AV/GRR dan Kappa dihitung otomatis, bukan field yang dapat diketik. Formula tidak bisa diedit pengguna.
9. **Referensi Attribute (agreed):** hanya **Supervisor** yang boleh membuat, mengubah dan memverifikasi Known Reference OK/NG, melalui subview terpisah. Operator tidak boleh melihat Known Reference pada studi yang belum selesai/masih blind; setelah Final, laporan hasil dapat dibaca sesuai hak History tanpa memberi izin mengubah referensi.
10. **Results first:** judgement dan warning harus tampak tanpa scrolling ke rincian formula, sedangkan formula/konstanta disembunyikan pada expandable details.
11. **Kebijakan keputusan:** Standard Preset + Configurable **Approved** Policy. Hanya Supervisor yang memilih/mengelola kebijakan yang sudah memiliki persetujuan Quality. Tanpa kriteria wajib yang sah, laporan menampilkan perhitungan dan **Review Required**, bukan status kelulusan otomatis.

## 2. Screen inventory dan navigasi

| ID | Screen | Tujuan utama | Peran |
| --- | --- | --- | --- |
| S01 | Login | Autentikasi individual | Public (login) |
| S02 | Dashboard | Ringkasan studi dan pintasan | Operator, Supervisor (sesuai scope) |
| S03 | MSA History | Cari/filter/buka histori semua studi; edit hanya berdasarkan hak akses | Operator, Supervisor |
| S04 | New MSA — Type Selection | Pilih Variable / Attribute | Operator, Supervisor |
| S05 | Study Setup | Metadata wajib + Advanced Details (collapsed) | Pemilik studi/Supervisor |
| S06 | Appraisers & Samples | Isi 3 appraiser + ID part/sampel | Pemilik studi/Supervisor |
| S07 | Variable Measurement Input | 90 angka, tiga tab appraiser | Pemilik studi/Supervisor |
| S08 | Attribute Assessment Input | 300 OK/NG pada 3 tab; restricted Known Reference subview | Pemilik studi/Supervisor; reference: kontrol tambahan |
| S09 | Calculation Results & Review | Judgement + warning, Submit for Approval; Supervisor Approve & Finalize | Pemilik studi/Supervisor, tergantung policy |
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
       │                        ├─ Submit for Approval (Operator)
       │                        └─ Supervisor Approve & Finalize
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

### 3.3.1 Approval & Report provenance (read-only except authorized assets)

| Field key | UI label | Type | Rule |
| --- | --- | --- | --- |
| `approval_status` | Approval Status | Submitted / Needs Correction / Approved | Workflow flag on Draft, then finalized approval state |
| `submitted_by`, `submitted_at` | Submitted By / At | Server audit | Recorded when owner requests approval |
| `approved_by_user_id` | Approved By | Authenticated Supervisor ID | Must match server actor performing Approve & Finalize |
| `approver_name_snapshot`, `approver_title_snapshot` | Approver Name / Role | Read-only snapshot | Capture at approval for immutable report |
| `approved_at` | Approval Date / Time | Server timestamp | Capture actual time on successful Finalize |
| `approval_note` | Approval / Review Notes | Supervisor text | Optional for clean study; required if policy demands exception evidence |
| `logo_asset_id` | Company Logo | Approved asset reference | Render in PDF; use no invented image |
| `signature_asset_id` | Authorized Supervisor Signature | Permission-controlled asset reference | Can only be attached to Supervisor-approved PDF; never permit Operator to upload/replace approver signature |
| `pdf_report_artifact_id` | Final PDF | Immutable artifact/version | Points to report generated for approved finalized revision |

**Asset policy:** Company logo/signature files can be supplied later; until formally provided and validated, previews show **pending-asset placeholder**, not a fake signed PDF. **Approve & Finalize may be held for report issuance** when policy requires signature asset but it is missing; do not silently invent signatures. Approval identity comes from authenticated Supervisor action, not from an editable PDF text field.

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

- **Primary:** Create New MSA, Continue My Draft, **Pending My Approval** queue for Supervisor; recent studies from **all accessible study history**.
- **Metrics:** total studies, Draft, Finalized, Approval Pending/Needs Correction and Review Required; lifecycle vs statistical result clearly separated.
- **Operator visibility (agreed):** counts and read-only listing span **all studies**. Editing or finalizing studies belonging to other users is not allowed. Restricted unfinalized Attribute reference values must never leak via dashboard statistics, previews or search.
- **Supervisor:** studies across users, worklist for Approve & Finalize, exception review.
- **Admin:** separate User Management; role does not automatically grant study access.

### S03 — MSA History

- **Columns:** Study ID, Study Name, Type, Part/Characteristic, Study Date, Owner, Lifecycle Status, Approval Status, Quality Result (when permitted), Updated At, Revision.
- **Search:** ID, name, part number/name, characteristic; filters include Variable/Attribute, lifecycle, approval status, date, owner.
- **Operator (agreed):** can **see every study in the history list**, not only own. For others, study detail is **read-only**; no editing, Finalize, revision or delete. Hide restricted draft reference/raw result details that could undermine blinded trials.
- **Own Draft:** Continue/Edit; other person's Draft: View permitted metadata/status only unless released for read-only result review.
- **Finalized study:** Operator can view complete approved report/PDF for every study, but cannot change it; sensitive references remain subject to separate exposure controls.
- **Supervisor:** edit/manage Draft, review submissions, make revisions and soft delete; application/server permission checks at every endpoint.
- **UX:** pagination; soft-deleted items hidden from active list; avoid treating Draft/Finalized (lifecycle) as Good/Bad (quality result).

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

**One study / three appraiser tabs (agreed).** Fifty sample IDs shared across three appraisers, each with two independently collected OK/NG trials. **300 decisions total**, irrespective of one combined paper or three separate forms.

| Sample ID | Trial 1 | Trial 2 |
| --- | --- | --- |
| 01 | OK / NG / blank | OK / NG / blank |
| 02–50 | OK / NG / blank | OK / NG / blank |

**Appraiser tabs (Operator owner / Supervisor study editor):**
- Appraiser A, B, C; progress per tab 0/100 and overall 0/300; Save Draft, paste-from-Excel, Next Appraiser.
- Editing is limited to owner Draft or Supervisor. **Being able to view all History never grants editing rights** on other Drafts.
- No Known Reference answer, derived correctness hint or locked-result tooltip in blinded input view.

**Restricted Known Reference subview (Supervisor-managed, agreed):**
- One shared 50-sample reference table, entered/edited/verified **only by Supervisor**; source and verification audit fields.
- Operator cannot open restricted subview, fetch unfinalized reference via API, or use report/History shortcuts to see it during blind classification.
- Supervisor verifies references and confirms blind trials are complete before comparison results are disclosed to affected appraisers. Editing reference after review changes calculation freshness and requires recalculation/reapproval; Finalized changes require new version.
- References cannot be auto-created from majority vote or set to OK when blank. Handle sample identity consistently throughout tabs and reference.

**Prerequisites:** verified 50 reference labels + all 300 decisions + complete setup + distinct appraiser identities before pooled Kappa vs Reference can be calculated. If Quality acceptance thresholds are not approved, the calculations can be displayed but **no automatic OK** granted. Distinguish statistical completeness from Quality policy approval.

**Visibility after Finalized:** released PDF and results may be viewed read-only from History by Operators; reference-management permissions remain exclusive to Supervisor. Do not leak unfinalized blinded source data in all-study History.

### S09 — Calculation Results & Review

**Information hierarchy:**
1. **Judgement:** method Calculated Band and approved-policy Overall Disposition are separate, both clearly labeled; missing policy → Review Required, not automatic OK.
2. **Warnings:** NDC, R-chart, missed-NG/false reject, undefined rates, missing reference/policy, stale results. Provide specific remediation.
3. **Key figures:** Variable %Study Var, NDC, %Tolerance; Attribute three appraiser rows: pooled Kappa, Kappa band, missed NG and false reject with denominators.
4. **Expandable Calculation Details** (default collapsed): formulas, constants, Rbar, EV/AV/GRR/PV/TV, per-subgroup ranges, Po/Pe, 2×2 confusion counts, additional agreement/consistency.
5. **Approval panel:** **Submit for Approval** button for owner (available after valid current results), status Submitted / Needs Correction / Approved, reviewer notes, actor/timestamp. **Approve & Finalize** is **Supervisor-only**.

**Variable-specific:** Ratama %Study Var bands <10% OK, 10–30% Special OK, >30% NG (method band); NDC ≥5, R-chart and %Tolerance (when LSL/USL valid) shown separately. Conditional/conflict/undefined results trigger Review Required. Do not invent a %Tolerance acceptance cutoff.

**Attribute-specific:** Ratama pooled Kappa **per appraiser**, not a global average, paired with accuracy, trial consistency, missed-NG C/(A+C), and false-reject B/(B+D). Known Reference is strictly protected during any active blind process. The whole-study decision follows only an **approved, versioned** Quality policy. Without one, metrics may be shown but overall disposition remains Review Required.

**Approved workflow design (product-owner confirmation; whole specification NOT FROZEN):**
1. Operator **Calculate → Save/Submit for Approval**; cannot Finalize, even for a clean result. Supervisor may also submit their own study for documented review, but an authenticated Supervisor approval event is always required.
2. Supervisor opens submitted study, reviews complete/current metrics, warnings, policy version, verified references and any corrections; may select **Request Correction** (state remains Draft) and record reason.
3. Supervisor may **Approve & Finalize** only if the configured policy authorizes the recorded disposition. The approval action is atomic with Finalize; no second manager/department tier.
4. At successful approval, server records approver user ID, displayed name/role snapshot, approval time, report/policy snapshot and final disposition; PDF displays **company logo + authorized approval signature**, along with approver name, job title and approved-on date.
5. If required Quality policy or risk disposition is unresolved, **do not approve a misleading OK**. Remain Review Required / Pending Approval or obtain explicit approved exception policy; any not-approved archival report must be labeled accurately.
6. Previous finalized revisions and their PDF/approval snapshots remain immutable.

**No cosmetic approval:** A pasted signature without a recorded Supervisor authorization never makes a study Finalized.

### S10 — Study Detail & Revisions

**Panels:** Overview, Input Data (read-only when Final), Results, Report, Approval History, Edit Audit, Revisions.
- **All Operator history:** all study entries searchable. Own study: Draft editable; other people's studies: read-only metadata/status during Draft; finalized reports are viewable/downloadable, never editable.
- **Show:** source appraiser separately from actor who entered data, submitted by/at, Supervisor approved by/at, policy/method versions and disposition, corrections and revision lineage.
- **Actions:** Continue own Draft; Submit/Withdraw before approval (with audit); Supervisor Approve, Request Correction, Create Revision and Soft Delete.
- **Final immutability:** Supervisor editing a finalized version is forbidden; Create Revision and preserve the approved PDF and signature record of previous version.

### S11 — Report Preview / PDF (logo + approval signature)

**Report header:** Company name and **approved company logo**; study ID/name, part, characteristic, method/version and revision. First page prioritizes **Overall Disposition**, warnings and principal statistics. Do not imply Quality conformity without approved policy.

**Approval block (required for official Final report):**
- Label **Approved by Supervisor** (not generic unsigned "Performed By").
- Approver name, job title/role, approval timestamp (with business timezone), approval record ID / audit provenance.
- **Supervisor's authorized signature** rendered from a permission-controlled, approved asset after explicit Supervisor **Approve & Finalize** action. Company logo asset appears in the designated header area.
- Assets can be supplied later. When missing, **show placeholder in design/draft preview**, never fabricate logo or signature; official report issuance must respect any applicable requirement to supply signed asset. Whether PDF includes uploaded wet-ink scan, drawn signature or e-signature requires a later security/legal implementation decision.
- Do **not** label an inserted image signature as cryptographically certified digital signature.

**Statistical contents:**
- Variable: study setup, %Study Var band, NDC, %Tolerance if defined, R-chart alerts, EV/AV/GRR/PV/TV and calculated detail.
- Attribute: appraiser-level pooled Kappa and source band, missed NG/false reject counts and rates, repeatability, study reference composition and policy/version; no unapproved global Kappa average.

**Actions:** Preview, Download Final PDF, Print. Draft preview, if enabled, is marked **DRAFT — NOT APPROVED** with no approval signature. Each finalized report uses its own stored immutable snapshot and original approved policy/signature provenance. Other Operators can access finalized reports read-only through History.

**Still open:** Ratama logo/signature files, precise signatory title, visual layout, full sample-reading appendix and whether a formally regulated e-signature standard applies.

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

## 5. Authorization matrix — accepted product-owner direction

| Feature | Admin | Supervisor | Operator |
| --- | --- | --- | --- |
| Login / own session | Yes | Yes | Yes |
| Manage accounts / roles | Yes | No | No |
| Dashboard / all-study History list | No study privilege by Admin role alone | All studies | **All studies (agreed)** |
| View other owner's Draft | No | Yes | **Read-only metadata/status; not blind reference** |
| View/download others' finalized reports | No | Yes | **Yes, read-only (agreed)** |
| Create study / edit own Draft | No | Yes | Yes |
| Edit other user's Draft | No | Yes | **No** |
| Transcribe Variable/Attribute measurements on own Draft | No | Yes | Yes (not restricted Known Reference) |
| Create/edit/verify Attribute Known Reference | No | **Yes, exclusive** | **No** |
| Calculate | No | Yes | Own study, with blind-results safeguards |
| Submit for Approval / Request Correction | No | Yes | Submit own only |
| **Approve & Finalize** | No | **Yes, exclusive** | **No** |
| Manage approved policy | No | With Quality approval evidence | No |
| Create revision / soft delete | No | Yes | No |
| Study audits | Account admin scope | All studies | Read-only applicable study history |
| Logo/signature asset authorization | Administrative upload mechanism subject to policy | Only approved signatory action applies signature | Cannot apply/rewrite approval signature |

**Access control:** Every endpoint must check role, action, study ownership, lifecycle, study publication/blind state and confidentiality restrictions. All-history visibility does **not** grant write permission or disclosure of active blind references. No public registration. **Supervisor-only approval and reference management, Operator visibility of all history are accepted design decisions but entire spec remains DRAFT**.

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
Draft --Calculate (all required source data valid)--> Draft + Current Result
Draft + input changed --> Draft + Stale Result (submit/approval invalidated)
Draft + Current Result --Owner Submit for Approval--> Draft + Approval Pending
Draft + Approval Pending --Supervisor Request Correction--> Draft + Needs Correction
Draft + Approval Pending --Supervisor Approve & Finalize (authorized policy)--> Finalized vN + Approved
Draft + Approval Pending --No approved criteria / unresolved risk--> Draft + Review Required (no automatic OK)
Finalized vN --Supervisor Create Revision--> Draft Revision vN+1
Finalized vN --Supervisor Soft Delete--> Hidden active listing; immutable audit retained
```

The review flags Approval Pending and Needs Correction are **workflow markers on Draft**, not new statistical categories or extra approval tiers. **Approve & Finalize** is one authenticated Supervisor event that records approver identity, timestamp, policy snapshot, method version and approval signature reference. No Operator can finalize or edit others' studies. An unresolved Quality judgment cannot be "approved OK" by merely attaching a signature. Previous Finalized versions and signed PDFs remain unchanged.

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
| UI-17 | Operator calculates a clean own study | Operator can Save/Submit for Approval but cannot Finalize; Supervisor queue receives it (agreed) |
| UI-18 | Supervisor finalizes with correct current results and an approved policy | Finalized snapshot records reviewer/time, formula/policy version and disposition |
| UI-19 | Supervisor sees no approved policy or unacceptable/undefined risk | No automatic OK; Review Required with documented exception handling |
| UI-20 | Operator is actively collecting Attribute blind trials | Neither UI nor API exposes Known Reference or correctness-derived signals |
| UI-21 | Three paper sheets arrive on separate dates | All decision values preserved in one study with independent appraiser tab progress |
| UI-22 | Results first loaded | Judgement and warnings visible immediately; formula and constants hidden until user expands Calculation Details |
| UI-23 | User returns from Results to change one input | Previously calculated result marked Stale, review submission invalidated until recalculated |
| UI-24 | Supervisor enters 50 verified reference values | Reference is not an appraiser tab; reference labels stay masked from unauthorized users |
| UI-25 | Policy gets updated after old study Finalized | Historical judgement and report stay exactly as frozen in prior study version |
| UI-26 | Operator opens History with other people's Drafts and Finalized studies | All study records appear; others' Draft metadata read-only, finalized PDFs downloadable |
| UI-27 | Operator attempts to modify another owner's Draft through direct API | Backend rejects write even though study appears in all-study History |
| UI-28 | Operator calls Known Reference endpoint on own study | Denied; only Supervisor can enter/edit/verify; no blind reference disclosed |
| UI-29 | Operator tries to Finalize own complete study | Denied; only Submit for Approval available; Supervisor approval queue updated |
| UI-30 | Supervisor approves a complete valid study | Single Approve & Finalize action records approver, role, time and immutable approved revision |
| UI-31 | PDF generated from Supervisor-approved study | Official PDF contains configured company logo, signatory name, title, approval date and authorized signature |
| UI-32 | Approval signature asset is not yet provided | No forged signature; report issuance follows missing-asset policy, draft clearly unsigned |
| UI-33 | Operator views others\' finalized report | Report read-only; cannot edit policy, signature, underlying study or previous revision |
| UI-34 | Supervisor rejects/requests correction | Draft remains editable by owner; reviewer, reason and time recorded; no Final state |
| UI-35 | Supervisor attempts approval with no approved Quality policy | No invented OK even if signature asset exists; remain Review Required |

## 9. MVP exclusions

No SPC, multi-client tenant management, automatic full-file Excel import, ANOVA/nested GRR, custom statistical formula editor, multi-level approval hierarchy, mandatory appraiser logins, parallel section-level multi-user editing, mobile-native app, broad notifications or Minitab output-clone UI.

## 10. Remaining implementation and domain decisions (not blockers for draft UX)

**Four earlier questions resolved by product-owner:**
1. Supervisor alone Approves & Finalizes.
2. Supervisor alone manages/edits/verifies Attribute Known Reference; active blind trials stay protected.
3. Operator may view all study history; other users' Drafts are read-only and active reference values protected, Finalized reports readable.
4. Official report will contain company logo, Supervisor approval record and authorized signature.

**Still pending:** Ratama-provided logo and signature assets; final signatory name/job title, signature asset authorization policy (scanned signature versus other form), PDF visual layout/appendix, customer/Quality-approved risk thresholds and special NG exception dispositions, and applicable legal requirements if certified e-signature is later desired. Actual acceptance testing and implementation have not begun.

This product-owner confirmation is **not** blanket approval to mark the entire document, statistical formulas or customer Quality policies FROZEN.

**Change control:** v0.1 remains in GitHub history and as a standalone file. No historical file is overwritten by v0.2.

**Document state:** Product design candidate only. Requires stakeholder review, QA, and domain approval; **NOT FROZEN**, no application code written.
