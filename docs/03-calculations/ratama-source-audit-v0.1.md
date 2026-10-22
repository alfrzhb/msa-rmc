# Ratama MSA Source Audit v0.1

**Status:** DRAFT — independently reviewed numerical outputs; NOT FROZEN  
**Date:** 2026-10-22  
**Owner:** RMC MSA Analysis Tool discovery  
**Scope:** Variable and Attribute reference workbooks, Ratama training presentation and supporting Attribute notes.

## 1. Purpose and source authority

The client's need is a simple, accurate MSA workflow—not a feature-complete Minitab clone. Ratama's training and source workbooks define the intended *business workflow and method family*. Recalculation must use mathematically valid formulas, even if the workbook's computed cells are incorrect.

Source materials inspected (filenames, not redistributing original client files):

1. **Form Analisa MSA Variabel_MDI.xlsx** — full 10 × 3 × 3 input grid and evaluated outputs.
2. **Form MSA ATTRIBUTE KAPPA_PT XYZ.xlsx** — 50 × 3 × 2 classifications, reference and crosstab.
3. **Modul Coretools_MSA_220930_TVS.pptx** — Ratama training deck (67 extracted slide sections), particularly slides 41, 45–48, 52, 56, 60 and 66.
4. **Atribut MSA _ 20 Juni 2025.docx** — Attribute interpretation and worked examples.
5. **RMC_MSA_Calculation_Specification_v1.0_FROZEN.docx** — *historical* specification, cross-check only; it is NOT the frozen baseline of this rebuilt project.

Independent public technical references:
- [Minitab — crossed GRR formulas](https://support.minitab.com/en-us/minitab/help-and-how-to/quality-and-process-improvement/measurement-system-analysis/how-to/gage-study/crossed-gage-r-r-study/methods-and-formulas/gage-r-r-table/)
- [Minitab — interpreting crossed GRR](https://support.minitab.com/minitab/help-and-how-to/quality-and-process-improvement/measurement-system-analysis/how-to/gage-study/crossed-gage-r-r-study/interpret-the-results/all-statistics-and-graphs/)
- [Minitab — Attribute Kappa](https://support.minitab.com/en-us/minitab/help-and-how-to/quality-and-process-improvement/measurement-system-analysis/how-to/attribute-agreement-analysis/attribute-agreement-analysis/methods-and-formulas/kappa-statistics/)
- [Minitab — interpreting Attribute Agreement](https://support.minitab.com/en-us/minitab/help-and-how-to/quality-and-process-improvement/measurement-system-analysis/how-to/attribute-agreement-analysis/attribute-agreement-analysis/interpret-the-results/key-results/)

**Verification boundary:** Source spreadsheet *displayed/evaluated data* were independently recomputed, and displayed worksheet formulas/output labels were cross-checked where visible. This audit does not claim to have inspected every underlying OOXML cell formula, named range, link, or hidden sheet. A cell-by-cell formula dependency and hidden-sheet audit remains a QA follow-up if the raw workbook is programmatically inspected.

## 2. Ratama source alignment and differences

| Topic | Ratama PPT | Ratama workbook | Implication |
| --- | --- | --- | --- |
| Variable part count | 10 parts in study walkthrough (slide 41) | 10 | 10 is a justified MVP preset |
| Variable operators | Follow actual measurement operators (slide 41) | 3 | 3 is a *preset*, not a universal rule |
| Variable trials | 2 or 3 (slide 41) | 3 | 3 is a justified MVP preset |
| Variable method | Range, Average & Range, ANOVA discussed (slide 45) | Average & Range | Start with Average & Range; do not claim ANOVA equivalence |
| Variable statistics | Study variation and tolerance are distinct (slide 56) | Mainly %GRR and NDC | Display both %Study Var and %Tolerance when specification exists |
| NDC | ≥5 (slide 52) | ≥5 | Align requirement, subject to validation |
| Attribute sample count | 20–30 parts in example (slide 60) | 50 | 50 is workbook-based MVP preset, not an absolute standard |
| Attribute trials | Inspectors assess repeated samples | 2 | 2 is workbook-based MVP preset |
| Attribute labels | OK/NG or Go/No-Go examples | OK and NG with known reference | Preserve binary labels and known-reference requirement |
| Process documentation | Study list, outcome, corrective action (slide 66) | Spreadsheet report | History and audit/reporting are justified |

**Source precedence proposal:** validated statistical method > Ratama intended study methodology > workbook layout/test data > historical draft documents. When there is disagreement about a customer-specific acceptance rule, escalate to Ratama/customer rather than invent a universal standard. The PPT also notes that alternative methods/criteria may require customer approval.

## 3. Variable workbook — independent reconciliation

Dataset: 10 parts × 3 appraisers × 3 trials = **90 numeric readings** in 30 operator–part subgroups. Characteristic: Gap Datum 1; specification: 15.5 ± 0.1 mm (LSL 15.4; USL 15.6; tolerance width 0.2 mm).

Constants copied from the provided Variable workbook for this specific configuration:
- K1 = 0.5908 (3 trials).
- K2 = 0.5231 (3 appraisers).
- K3 = 0.3146 (10 parts).
- D4 = 2.58 (3-trial R-chart limit; rounded reference coefficient).

| Statistic | Independently recomputed | Notes |
| --- | ---: | --- |
| Mean subgroup range Rbar | 0.0040000000 | Average of 30 subgroup ranges |
| XDIFF | approximately 0 | Floating point difference ≈ 1.8e-15 |
| Range of part averages Rp | 0.0766666667 | Across 10 part means |
| Repeatability EV | 0.0023632000 mm | Rbar × K1 |
| Appraiser Variation AV | 0 mm | Square-root argument negative; clamp to zero |
| GRR | 0.0023632000 mm | sqrt(EV² + AV²) |
| Part Variation PV | 0.0241193333 mm | Rp × K3 |
| Total Variation TV | 0.0242348294 mm | sqrt(GRR² + PV²) |
| %GRR of Study Variation | **9.751255%** | 100 × GRR / TV |
| %GRR of Tolerance | **7.089600%** | 100 × (6 × GRR) / 0.2 mm |
| NDC raw | 14.390767 | 1.41 × PV / GRR |
| NDC displayed | **14** | Integer truncation toward zero for positive result |
| R-chart UCL | **0.0103200 mm** | Rbar × 2.58 |
| Max observed subgroup range | 0.0100000 mm | No range above UCL |

**Observed workbook discrepancies:**
1. Workbook displays **AV = −0.0004314593**; a variability standard deviation must not be negative. Use **AV = sqrt(max(0, (XDIFF × K2)² − EV²/(parts × trials))) = 0**.
2. This erroneous AV propagates into workbook **GRR ≈ 0.0024022638** and **%GRR ≈ 9.910873%**; independently corrected %GRR is **9.751255%**.
3. Workbook displays **UCLR ≈ 39.99688**. This corresponds to using the grand *measurement mean* (~15.50267) rather than Rbar (~0.004) for the R-chart limit. Correct UCL for the given D4 is **0.01032**.
4. Some workbook percentage output cells show *fractions* (e.g., ~0.0991) near labels that imply percentages; enforce explicit percentage units and formatting.

**Note:** These are observable computation/output discrepancies, not proof of the exact wrong OOXML formula address. The dataset is the reference, while the erroneous displayed outputs are not valid golden values.

## 4. Attribute workbook — independent reconciliation

Dataset: **50 samples**, **3 appraisers**, **2 trials each**, **300 judgments**. Known reference has **14 NG** and **36 OK** distinct samples. Each appraiser makes 100 judgments against the repeated reference: **28 reference-NG and 72 reference-OK decisions**.

Let A = appraiser NG / reference NG; B = appraiser NG / reference OK; C = appraiser OK / reference NG (**missed NG**); D = appraiser OK / reference OK.

| Appraiser (anonymized order) | A | B | C | D | Po | Pe | Correct Kappa | Original workbook Kappa |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| 1 | 15 | 1 | 13 | 71 | 0.86 | 0.6496 | **0.600457** | 0.618096 |
| 2 | 16 | 0 | 12 | 72 | 0.88 | 0.6496 | **0.657534** | 0.676851 |
| 3 | 11 | 4 | 17 | 68 | 0.79 | 0.6540 | **0.393064** | 0.404762 |

Formulas: N=A+B+C+D; Po=(A+D)/N; Pe=((A+B)(A+C)+(C+D)(B+D))/N²; Kappa=(Po−Pe)/(1−Pe).

The original worksheet result for **appraiser 3 crosses the 0.40 boundary**, from **0.404762 (Improvement)** to **0.393064 (Poor)** after correction. This changes the qualitative interpretation—not just decimal precision.

### 4.1 Important inspection-safety finding

High overall agreement **does not** guarantee an inspector detects NG samples effectively.

| Appraiser | Observed agreement | Within-appraiser match (same sample across 2 trials) | Missed NG (C / reference NG decisions) | False reject (B / reference OK decisions) |
| --- | ---: | ---: | ---: | ---: |
| 1 | 86% | 46/50 = 92% | **13/28 = 46.43%** | 1/72 = 1.39% |
| 2 | 88% | 50/50 = 100% | **12/28 = 42.86%** | 0/72 = 0% |
| 3 | 79% | 47/50 = 94% | **17/28 = 60.71%** | 4/72 = 5.56% |

Definitions:
- Missed NG rate = reference NG classified OK / all reference NG judgments.
- False reject rate = reference OK classified NG / all reference OK judgments.
- These are **class-conditional** rates; dividing by *all* decisions would measure a different quantity.
- Within-appraiser match means the two trial labels agree; **agreement need not mean correctness**.
- Treat these rates as risk indicators; **do not silently replace Ratama Kappa judgment thresholds**.

Because the same 50 samples are assessed twice, 100 appraiser decisions per appraiser are **not 100 independent unique samples**. Descriptive pooled Kappa matches the workbook approach; any uncertainty interval or significance claim needs a method that accounts for clustering by sample.

### 4.2 Attribute threshold and guidance mismatches

The workbook's written middle-band notation resembling **0.4 ≥ Kappa < 0.75** is logically malformed. Historical Ratama specification and supporting Attribute notes intend **0.40 ≤ Kappa < 0.75**; with **Kappa ≥ 0.75** = OK, **Kappa < 0.40** = Poor. The boundary at 0.75 is subject to explicit domain approval, as the training narrative uses slightly different comparison wording.

Ratama's separate Attribute notes also discuss effectiveness, false alarm and miss rates, but sample examples and denominator descriptions are not fully consistent. These should be **redefined explicitly** before they are elevated to final acceptance criteria.

## 5. Source-related risk register

| Severity | Risk | Containment |
| --- | --- | --- |
| Critical | Workbook Kappa error changes appraiser 3's decision class | Independently compute Po, Pe, Kappa; regression test class boundaries |
| High | Missed NG rates >40% despite relatively high agreement | Show class-conditional safety warnings; seek customer safety criteria |
| High | Negative AV incorrectly influences GRR | Clamp variance estimate to zero before square root; test |
| High | Incorrect R-chart UCL hides out-of-control ranges | Use Rbar, not measurement mean; test |
| High | Assumptions for crossed, repeatable measurements may not hold | Require study prerequisites and operator guidance |
| Medium | PPT and workbook differ on study size | Use workbook sizes as preset, not universal methodology |
| Medium | Percent-of-variation and percent-of-tolerance conflated | Clearly label and separately calculate |
| Medium | Kappa repeated observations treated as fully independent for confidence statements | Report descriptive statistics; do not assert unvalidated CIs |
| Medium | Fixed 6σ vs other study-multiplier settings | Follow Ratama slide 56: 6 × SD, versioned |
| Medium | Source data may be confidential | Preserve original workbook in authorized storage, **not public GitHub** |

## 6. Audit status

**Verified:** 90 Variable source readings recalculated; 300 Attribute source judgments and 3 crosstabs recalculated; Ratama presentation methods and study design contrasted; source output discrepancies identified.

**NOT yet verified / approval pending:**
- Formula dependency check of every raw XLSX cell, hidden sheet, and named range.
- Ratama/client final signoff for method use and acceptance thresholds.
- Operational consequences of class-conditional Attribute error rates.
- Alternative study size constant tables and statistical validity.
- Independent executable regression tests within the future application repository.

**No source workbooks, appraiser identities, or full row-level client measurements were committed.**
