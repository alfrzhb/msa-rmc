# Final Calculation Review — Variable & Attribute MSA v0.2

**Status: REVIEW DRAFT — NOT FROZEN**  
**Review date:** 2026-10-22  
**Repository:** alfrzhb/msa-rmc  
**Purpose:** Consolidate formula reconciliation, method choice, acceptance logic and outstanding approval gates before a formal calculation specification baseline.

> **Approval rule:** The user's request to review does NOT authorize changing document status to FROZEN. The current report is a proposed baseline for stakeholder review, not a validated certification claim.

## 1. Sources and method hierarchy

Primary Ratama evidence:
- `Form Analisa MSA Variabel_MDI.xlsx`, 10 parts × 3 appraisers × 3 trials.
- `Form MSA ATTRIBUTE KAPPA_PT XYZ.xlsx`, 50 samples × 3 appraisers × 2 trials.
- `Modul Coretools_MSA_220930_TVS.pptx` (slides 41, 45–48, 52, 56, 60, 66).
- `Atribut MSA _ 20 Juni 2025.docx` (Attribute guidance).
- Historical calculation spec v1.0 is informative, not a frozen baseline of this rebuild.

Independent methodological references:
1. [Minitab, Crossed Gage R&R methods and formulas](https://support.minitab.com/en-us/minitab/help-and-how-to/quality-and-process-improvement/measurement-system-analysis/how-to/gage-study/crossed-gage-r-r-study/methods-and-formulas/gage-r-r-table/)
2. [Minitab, NDC formula/display](https://support.minitab.com/en-us/minitab/help-and-how-to/quality-and-process-improvement/measurement-system-analysis/how-to/gage-study/crossed-gage-r-r-study/methods-and-formulas/number-of-distinct-categories/)
3. [Minitab, Gage acceptance guidelines](https://support.minitab.com/en-us/minitab/help-and-how-to/quality-and-process-improvement/measurement-system-analysis/supporting-topics/gage-r-r-and-wheeler-s-emp-studies/is-my-measurement-system-acceptable/)
4. [Minitab, Kappa with a known standard](https://support.minitab.com/en-us/minitab/help-and-how-to/quality-and-process-improvement/measurement-system-analysis/how-to/attribute-agreement-analysis/attribute-agreement-analysis/methods-and-formulas/kappa-statistics/)
5. [Minitab, assessment-agreement counting convention](https://support.minitab.com/en-us/minitab/help-and-how-to/quality-and-process-improvement/measurement-system-analysis/how-to/attribute-agreement-analysis/attribute-agreement-analysis/interpret-the-results/all-statistics-and-graphs/assessment-agreement/)
6. [Minitab, Assistant vs Stat menu differences](https://support.minitab.com/en-us/minitab/help-and-how-to/quality-and-process-improvement/measurement-system-analysis/supporting-topics/attribute-agreement-analysis/attribute-agreement-analysis-in-assistant-versus-stat-menu/)

**Rule:** Preserve Ratama's selected *study approach*, independently correct faulty workbook arithmetic, and never claim exact Minitab output equivalence if aggregation/conventions differ. Customer-specific alternate acceptance criteria require explicit domain approval.

## 2. Review verdicts

| Area | Verdict | Status |
| --- | --- | --- |
| Business intent | Simple login → create MSA → input → calculate → history/PDF | Previously agreed, not FROZEN |
| Variable method | Crossed Gage R&R Average & Range using 10×3×3 initial preset | Direction approved |
| Variable mathematics | Rbar→EV→AV→GRR→PV→TV→%Study Var→NDC; R-chart; %Tolerance | Recomputed, review-ready |
| Attribute method | Pooled binary Cohen-style Kappa for each appraiser vs fixed reference, 50×3×2 preset | Candidate approved by user for final review |
| Attribute support metrics | Accuracy, within-appraiser match, missed NG, false reject | Candidate approved by user for final review |
| Automatic overall acceptance policy | Requires policy for conflicting indicators and conditional bands | OPEN |
| Broader study configurations | Not authorized until constants, methods and fixtures validated | OPEN |
| Detailed raw XLSX formula-cell dependency inspection | Not yet performed | QA follow-up |

## 3. Variable formula contract (10×3×3 only)

Notation: n=10 parts, a=3 appraisers, r=3 trials; x(i,j,k) measurement; R(i,j)=max over repeated trials − min over repeated trials; Rbar = mean of n×a subgroup ranges; XDIFF = range across appraiser means; Rp = range across part means.

**Audited Ratama preset constants:** K1=0.5908; K2=0.5231; K3=0.3146; D4=2.58 (rounded; R-chart); study-variation multiplier=6.

1. EV = Rbar × K1.
2. AV = sqrt(max(0, (XDIFF × K2)² − EV²/(n×r))).
3. GRR = sqrt(EV² + AV²).
4. PV = Rp × K3.
5. TV = sqrt(GRR² + PV²).
6. %Study Var = 100×GRR/TV, defined only for TV>0.
7. %Tolerance = 100×(6×GRR)/(USL−LSL), defined only for valid paired specification limits (USL>LSL).
8. NDCraw=1.41×PV/GRR when GRR>0; integer NDC: truncation toward zero, with an OPEN boundary convention: Minitab displays at least 1 if positive NDCraw<1, while simple Ratama floor would display 0.
9. R-chart UCL=D4×Rbar; LCL=0 for this 3-trial preset; flag subgroup ranges above UCL.

Notes:
- %Study Var, %Tolerance, and variance component %Contribution are **different metrics**, not interchangeable.
- A different count of parts/appraisers/trials requires new verified constants and regression fixtures.
- Calculate from full-precision values; apply rounding only in UI and report.
- Correct invalid AV and UCL arithmetic inherited from original spreadsheet, not the method itself.
- No standalone bias, stability, linearity, nested/destructive studies or ANOVA in MVP.
- Sample representativeness, measurement resolution and properly randomized repeated measurement are necessary study prerequisites, not consequences of passing numeric thresholds.

### Variable regression values (original 90 observations)

| Metric | Expected |
| --- | ---: |
| Rbar | 0.0040000000 |
| XDIFF | approx. 0 (floating residual ~1.8e−15) |
| EV | 0.0023632000 |
| AV | 0 |
| GRR | 0.0023632000 |
| PV | 0.0241193333 |
| TV | 0.0242348294 |
| %Study Var | **9.7512549552%** |
| %Tolerance (LSL=15.4, USL=15.6 mm) | **7.0896000000%** |
| NDCraw | 14.3907667569 |
| NDC integer | **14** |
| R-chart UCL | **0.0103200 mm** |
| Ranges above UCL | **0** |

The source workbook's displayed negative AV, ~9.910873% GRR, and ~39.99688 R-chart UCL are not accepted as correct outputs.

### Variable judgment candidate

- %Study Var GRR <10% ⇒ source label OK.
- 10%≤%Study Var GRR≤30% ⇒ Conditional / Special OK (**not** unconditional pass).
- >30% ⇒ source label NG.
- NDC≥5 is the candidate distinction criterion.
- R-chart alarms must produce investigation guidance; they must not be suppressed by a favorable aggregate %GRR.
- %Tolerance must be shown separately when applicable; whether it controls a final decision is an explicit OPEN policy decision.
- **Proposed UI behavior:** display %Study Var band, NDC status, %Tolerance, and chart-warning status independently; if contradictory/unsafe, show **Review Required** until supervisor resolution. Never invent an unapproved combined threshold.

## 4. Attribute formula contract (50×3×2 preset)

Each sample has exactly one known OK/NG reference. Each appraiser records two OK/NG decisions for each of 50 samples; per appraiser **100 decisions** but only **50 distinct physical samples**.

For each appraiser, pooled across both trials:

| | Reference NG | Reference OK |
| --- | ---: | ---: |
| Appraiser NG | A | B (false reject) |
| Appraiser OK | C (missed NG) | D |

Let N=A+B+C+D=100; Po=(A+D)/N; Pe=((A+B)(A+C)+(C+D)(B+D))/N²; Kappa=(Po−Pe)/(1−Pe) if Pe<1.

Additional descriptive metrics:
- Decision-level accuracy = 100×(A+D)/N.
- Within-appraiser same-label match = samples with Trial1=Trial2 / 50 (not correctness).
- All-trials-correct sample percentage = samples where **both** judgments match reference / 50.
- Missed NG rate = 100×C/(A+C), conditioned on reference NG.
- False reject rate = 100×B/(B+D), conditioned on reference OK.
- Optional pairwise/between-appraiser indicators require separate precise label/method documentation; do not imply they equal multi-rater overall Kappa.

### Attribute regression values

Reference breakdown: 14 NG / 36 OK distinct samples; 28 NG and 72 OK *decision-reference comparisons* per appraiser.

| Appraiser | A / B / C / D | Pooled Kappa | Decision accuracy | Same-label match | Both trials correct | Missed NG | False reject |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: |
| 1 | 15/1/13/71 | **0.6004566210** | 86% | 92% | **82%** | 46.43% | 1.39% |
| 2 | 16/0/12/72 | **0.6575342466** | 88% | 100% | **88%** | 42.86% | 0% |
| 3 | 11/4/17/68 | **0.3930635838** | 79% | 94% | **76%** | 60.71% | 5.56% |

Appraiser 3's erroneous workbook Kappa is **0.4047619**; the corrected **0.3930636** falls below the candidate 0.40 judgment boundary. This is a substantive category change.

### Critical: pooled Ratama Kappa is **not** the Minitab known-standard method

Minitab's documented known-standard Kappa first computes the **Kappa of each trial against the reference, then averages the trial-level coefficients**. Ratama's workbook first pools the two trials into one crosstab and computes one coefficient. These are not algebraically interchangeable.

| Appraiser | Ratama pooled Kappa | Trial 1 Kappa | Trial 2 Kappa | Mean trial Kappa |
| --- | ---: | ---: | ---: | ---: |
| 1 | 0.6004566210 | 0.6102449889 | 0.5901639344 | 0.6002044616 |
| 2 | 0.6575342466 | 0.6575342466 | 0.6575342466 | 0.6575342466 |
| 3 | 0.3930635838 | 0.2788461538 | 0.4988864143 | 0.3888662841 |

**Product recommendation:** retain **Ratama Pooled Kappa vs Reference** as *versioned primary* MVP output, explicitly labeled as such. Do not claim numbers are identical to Minitab Attribute Agreement Analysis. Store raw trial-level outcomes so a later separately versioned trial-mean method can be supported if requested.

### Attribute judgment candidate

- Kappa≥0.75 ⇒ OK.
- 0.40≤Kappa<0.75 ⇒ Improvement.
- Kappa<0.40 ⇒ Poor.
- Worksheet printed middle inequality is malformed; the preceding is the coherent interval inferred from the intended bands. Exact boundary conventions still require domain approval.
- Missing NG / OK classes make corresponding conditional error rates **Not Calculable**; if Pe=1, Kappa **Not Calculable**.
- **Do not** base system-wide OK only on overall decision agreement or mean Kappa: sample OK/NG imbalance can hide missed NG.
- Appraiser-level indicators and any overall status must remain distinguishable.

## 5. Statistics that must not be mislabeled

1. **Decision-level accuracy** (e.g., 86% from 86/100) is not the **all-trials-correct sample agreement** (e.g., 82% from 41/50). Minitab's Stat-menu Each-Appraiser-vs-Standard assessment table uses the latter counting rule; Minitab's Assistant route may count decision-level events differently.
2. **Within-appraiser repeatability** (92% same labels) is not the same as **accuracy** (86% per decision).
3. **Pooled Kappa** is not **mean trial Kappa**.
4. **Missed NG** C/(A+C) is not the proportion of missed-NG outcomes among **all** decisions; the denominators and purpose differ.
5. **%Study Variation** and **%Tolerance** are not interchangeable.
6. **Kappa** alone does not prove the measurement system safe for any particular defect type. Ratama/customer must determine acceptable risk for critical product characteristics.

## 6. Data integrity & statistical review gates

### Variable
- One unique numeric reading per part × appraiser × trial; 90 required for the preset.
- Enforce all appraisers measured the **same** parts using a valid crossed design; identify cases requiring another method.
- Draft accepts incomplete data; Calculate requires complete, finite data.
- Tolerance percentages computed only with compatible measurement units and valid USL/LSL.
- Edge cases: AV²<0 ⇒ 0; GRR=0 ⇒ NDC Not Calculable; TV=0 ⇒ %StudyVar Not Calculable; range chart zeros/alarms must be interpreted rather than auto-passed.
- Use full unrounded numbers to compare 10%/30% and NDC threshold.

### Attribute
- One reference value for each of 50 distinct samples; 2 valid binary appraisals per sample and appraiser; no missing decisions at Finalize.
- Reference should not be shown to appraisers during blind inspection; repeated order should be randomized when operationally possible.
- Preserve reference counts to avoid spurious interpretations from severely unbalanced samples.
- Pe=1 ⇒ Kappa Not Calculable; do not fabricate perfect Kappa.
- Missing reference classes ⇒ corresponding error rate Not Calculable; accuracy alone should not override missing coverage.
- Compare unrounded Kappa with 0.40 and 0.75; no tolerance for arbitrarily shifting cutoff.
- Do not claim ordinary confidence intervals from 100 independent parts: repeated appraisals share the same 50 parts.

### Traceability
- Persist study ID, calculation version, method, study design, audit timestamp, input revision, constants, precision policy, reference categories, raw data and outputs.
- On input edit, invalidate previous calculated result and recompute before Finalize.
- Preserve immutable finalized versions and their reports, even after supervised revision or soft delete.

## 7. Recommendation package awaiting final review

1. **Variable:** Accept Ratama Average & Range 10×3×3 with corrected EV/AV/GRR/PV/TV, %Study Var, %Tolerance, NDC and R-chart. Keep %Study Var and %Tolerance as separate displayed assessments.
2. **Variable decision:** Retain Ratama 10/30% bands and NDC≥5; mark Conditional or conflicting/suspicious results as **Review Required** pending explicit supervisor confirmation; no automatic certification.
3. **Attribute:** Accept Ratama **pooled** appraiser-vs-reference Kappa 50×3×2 as named/versioned MVP method; publish accompanying decision-level accuracy, within-appraiser match, all-trials-correct match, missed NG and false reject.
4. **Attribute decision:** Retain candidate 0.40/0.75 Ratama bands per appraiser; retain NG-miss warnings, but do **not** invent NG error acceptance thresholds or global pass criteria.
5. **Configuration:** Ship only workbook-compatible presets initially; architecture can later add validated variants without silently reusing constants.
6. **NDC low-end:** Adopt Minitab-style min=1 when GRR>0 and NDCraw<1, or explicitly retain workbook floor=0; choose and version one behavior before implementation.
7. **Future QA:** Obtain Ratama/client method-and-criteria approval, inspect underlying XLSX formulas if a cell-specific provenance audit is necessary, and implement reproducible automated test fixtures.

## 8. Remaining approval gates — explicitly NOT FROZEN

| Gate | Decision or evidence needed | Proposed owner |
| --- | --- | --- |
| G1 | Ratama/customer endorses the use of **pooled** Kappa rather than Minitab trial-average as the primary Attribute number | Ratama/client quality lead |
| G2 | Final Variable combined judgement including %Tolerance and R-chart warnings, plus how Conditional is resolved | Ratama/client quality lead |
| G3 | Appraiser-level vs global Attribute decision and acceptable missed-NG / false-reject limits | Ratama/client quality lead |
| G4 | Threshold boundaries, NDC low-end (0 vs min 1), consistent labels/rounding in report | Product/domain reviewer |
| G5 | Confirm crossed sampling, actual gauge resolution, reference provenance, blind study protocol and subject-matter acceptance | Ratama/client quality lead |
| G6 | Golden figures, negative AV/Pe=1/zero-variation tests and calculation-version identifier approved for implementation | Engineering QA + domain reviewer |

**Release state:** Formula reconciliation is review-ready, but the method/decision policy is not yet approved as a final release specification. **Nothing in this report is FROZEN.**
