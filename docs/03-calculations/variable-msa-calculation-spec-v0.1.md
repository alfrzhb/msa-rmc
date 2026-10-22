# Variable MSA — Calculation Specification v0.1

**Status: DRAFT — accepted method direction, formula outputs independently checked; NOT FROZEN**  
**Last audit:** 2026-10-22  
**Basis:** Ratama Variable workbook, Ratama training deck, historical v1.0 reference, and independent Minitab formula documentation.

## 1. Agreed product direction

- Implement **Crossed Gage R&R, Average & Range (Xbar-R)** for the first Variable MVP.
- Preset: **10 parts × 3 appraisers × 3 trials = 90 readings** (the actual Ratama example).
- Keep study setup simple for operators; do not expose ANOVA/advanced method menus in MVP.
- Display **%Study Variation**, **%Tolerance (where tolerance is defined)**, **NDC**, and an **R-chart consistency indicator**. Include EV, AV, GRR, PV and TV in detailed results/PDF.
- Preserve the **intent and criteria** of Ratama's training/workbook while replacing invalid worksheet arithmetic with verified calculations.
- These product choices were agreed as the *direction* for MVP, not frozen final acceptance rules.

## 2. Study prerequisites and meaning

- **Crossed**: all appraisers repeatedly measure the **same identifiable parts**, normally non-destructively, with the same instrument/method and meaningful operating conditions.
- Actual trained appraisers should reflect process personnel. Use numbered parts, calibrated measurement equipment and preferably randomized/blinded measurement order (Ratama training deck slide 41).
- The selected 10 parts should reflect the variation relevant to the system's intended use. Do not claim that arbitrary part selection proves process capability.
- Variable GRR studies do **not** by themselves establish bias, linearity, stability or calibration compliance.
- If measurements are destructive or different parts are substituted per trial, the crossed design is inappropriate; block or redirect the study until an appropriate method is defined.

## 3. Input and notation

- Number of parts **n = 10**, appraisers **a = 3**, repeated trials **r = 3** for MVP.
- **x(i,j,k)**: numeric measurement for part i, appraiser j, trial k, all in the same unit.
- **R(i,j)** = max over trials x(i,j,k) − min over trials x(i,j,k).
- **Rbar** = mean of all n×a subgroup ranges.
- **Xbar(j)** = mean across all parts and trials measured by appraiser j.
- **XDIFF** = max_j Xbar(j) − min_j Xbar(j).
- **PartMean(i)** = mean across appraisers and trials for part i.
- **Rp** = max_i PartMean(i) − min_i PartMean(i).

One datum for each of the 90 part/appraiser/trial slots; duplicates, missing entries or nonnumeric values must fail pre-calculation validation. Drafts can remain incomplete.

## 4. Constants (only for the agreed preset)

| Name | Value | Interpretation |
| --- | ---: | --- |
| K1 | **0.5908** | Repeatability constant, r=3 |
| K2 | **0.5231** | Appraiser constant, a=3 |
| K3 | **0.3146** | Part constant, n=10 |
| D4 | **2.58** | Rounded UCL factor for R-chart with 3 replicates |
| Study variation multiplier | **6** | Ratama deck slide 56 uses 6 × SD |

**Never reuse these constants for a different n, a or r without a newly verified constant table/method.** Internal computation preserves full numeric precision; round only for display.

## 5. Calculation pipeline

1. **EV** = Rbar × K1.
2. **AV² candidate** = (XDIFF × K2)² − EV² / (n × r).
3. **AV** = sqrt(max(0, AV² candidate)).
4. **GRR** = sqrt(EV² + AV²).
5. **PV** = Rp × K3.
6. **TV** = sqrt(GRR² + PV²).
7. **%GRR Study Variation** = 100 × GRR / TV; only if TV > 0.
8. **NDC raw** = 1.41 × PV / GRR; only if GRR > 0.
9. **NDC integer** = floor(NDC raw), because the ratio is nonnegative.
10. **%GRR Tolerance** = 100 × (6 × GRR) / (USL − LSL), only when both limits are valid with USL > LSL.
11. **R-chart UCL** = Rbar × D4; for 3 replicates, LCL=0. Flag any subgroup range above UCL.
12. Optional detailed ratios: %EV = 100 × EV/TV; %AV = 100 × AV/TV; %PV = 100 × PV/TV.

Distinct percentages:
- **%Study Var** compares GRR against observed total GRR + part variation.
- **%Tolerance** compares 6σ measurement variation against engineering specification width.
- Do not confuse either with **%Contribution**, which uses variance components.
- Do not use the multiplier of 6 in the numerator of %Study Var when both numerator and denominator already share the same multiplier: it cancels.

## 6. Independently recomputed test vector

Ratama Variable workbook references *Gap Datum 1*, specification **15.5 ± 0.1 mm** (LSL=15.4; USL=15.6). Outputs based on all 90 displayed numeric readings:

| Metric | Expected (rounded) |
| --- | ---: |
| Rbar | 0.0040000000 |
| XDIFF | 0 (numerical tolerance, source floating-point residue ≈ 1.8e-15) |
| Rp | 0.0766666667 |
| EV | 0.0023632000 mm |
| AV | 0 mm |
| GRR | 0.0023632000 mm |
| PV | 0.0241193333 mm |
| TV | 0.0242348294 mm |
| %GRR Study Var | **9.751255%** |
| %GRR Tolerance | **7.089600%** |
| NDC raw | 14.390767 |
| NDC integer | **14** |
| R-chart UCL | **0.0103200 mm** |
| Largest R(i,j) | 0.0100000 mm |
| R-chart subgroup alarms | **0** |

The worksheet's reported GRR ≈ 0.0024022638 / 9.910873% and UCLR ≈ 39.99688 are **not accepted** as correct expected outputs. See [audit](ratama-source-audit-v0.1.md).

## 7. Judgment rules

Ratama's workbook and presentation support:
- %GRR **< 10%**: OK/acceptable candidate.
- **10% ≤ %GRR ≤ 30%**: SPECIAL OK / conditional — requires explanation and potentially customer approval, **not automatic OK**.
- %GRR **> 30%**: NG/unacceptable candidate.
- **NDC ≥ 5**: discrimination criterion met; NDC < 5: failed discrimination criterion.
- **R-chart signals**: investigatory warning, even when %GRR looks favorable.
- **%Tolerance**: show as separate relevant engineering metric; any final pass/fail policy needs Ratama/client review, including whether it controls the decision.

**Proposed composite UI, pending approval:** primary %GRR band + separate NDC + separate R-chart condition. If indicators conflict, show **Review Required**, never unconditionally output OK. Avoid automatically treating this composite state as an AIAG certificate.

## 8. Determinism, numerical stability and validations

- Reject missing data at Calculate, nonnumeric/NaN/Infinity inputs, incorrect measurement counts, and any invalid unit/tolerance definition.
- Preserve zero and negative raw numeric measurements as valid where scientifically meaningful; do not confuse negative measurement values with invalid negative *variance*.
- If AV² candidate is negative: set AV=0, but log this branch for audit.
- If GRR = 0: NDC = **Not Calculable (zero measured GRR)**; never silently return Infinity or a fabricated integer. Preserve the underlying data for domain review.
- If TV = 0: %Study Var = **Not Calculable (zero total variation)**.
- If tolerance width ≤ 0 or missing: %Tolerance = **Not Calculable** (not zero).
- If Rbar = 0: R-chart UCL = 0 and interpret with care, especially under rounding/resolution limits.
- All threshold comparisons use unrounded values; round only for UI/PDF.
- Use stable decimal input handling appropriate to instrument resolution; small floating point residues alone do not establish true appraiser differences.
- Every result stores **method identifier, version, constants/version, number of parts/appraisers/trials, units, time of computation, and revision ID**.

## 9. Still open — must not be treated as frozen

1. Ratama/customer approval of composite OK vs Review Required logic and %Tolerance decision priority.
2. Actual process applicability (crossed study assumptions, gauge resolution, sampling representativeness).
3. Error handling for UCL flags and operator actions.
4. Any later alternative sample sizes and corresponding constants/tables.
5. Final PDF details, rounding/display precision and test fixture approval.
6. Direct formula-cell dependency inspection of original xlsx if required.

**No ANOVA, nested/destructive GRR, bias/linearity/stability implementation is specified by this document.**
