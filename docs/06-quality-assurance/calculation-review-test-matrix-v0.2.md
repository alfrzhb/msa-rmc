# Calculation Review Regression Matrix v0.2

**Status:** REVIEW DRAFT — NOT FROZEN  
**Date:** 2026-10-22  
**Companion:** [Final Calculation Review v0.2](../03-calculations/final-calculation-review-v0.2.md)

This is an additional **design/QA test matrix**, not an executed application test suite. The numbers were recomputed during the study audit.

## 1. Variable numerical baseline

Ratama 10×3×3 data: 90 readings, Rbar=0.004; EV=0.0023632; AV=0; GRR=0.0023632; PV≈0.0241193333; TV≈0.0242348294; %StudyVar≈9.7512549552%; %Tolerance=7.0896%; NDC=14; R-chart UCL=0.01032.

Required regressions:
- Negative **AV² candidate** must lead to AV=0, not imaginary or negative variation.
- Source worksheet UCLR≈39.99688 and %GRR≈9.910873 must **fail** corrected golden tests.
- A true XDIFF=0 must not generate spurious operator reproducibility from 1e−15 floating artifacts.
- Boundary predicates: <10, exactly 10, exactly 30, >30; %Tolerance and %Study Var are separately named.
- NDC raw <1: choose explicit convention (Minitab minimum 1 vs Ratama literal floor 0), then test. NDC raw 5.999 truncates to 5, not 6.
- GRR=0 / TV=0 / missing or reversed specification limits / wrong units return unambiguous Not Calculable, never NaN/Infinity.
- An out-of-limit range produces a clear warning independent of %Study Var verdict.
- Out-of-scope destructive/nested study parameters must not be silently analyzed as a crossed design.

## 2. Attribute crosstab / source parity

References: 50 unique samples (14 NG and 36 OK). Each of three appraisers judges every sample twice, totaling 300 judgments.

| Appraiser | Pooled confusion A/B/C/D | Ratama pooled Kappa | Trial 1 Kappa | Trial 2 Kappa | Trial average |
| --- | --- | ---: | ---: | ---: | ---: |
| 1 | 15/1/13/71 | 0.6004566210 | 0.6102449889 | 0.5901639344 | 0.6002044616 |
| 2 | 16/0/12/72 | 0.6575342466 | 0.6575342466 | 0.6575342466 | 0.6575342466 |
| 3 | 11/4/17/68 | 0.3930635838 | 0.2788461538 | 0.4988864143 | 0.3888662841 |

**Gates:** Ratama pooled Kappa is the intended primary candidate; Minitab known-standard Kappa computes a mean of per-trial Kappas. The engine must not silently switch or compare the algorithms as if equivalent.

## 3. Attribute counting-convention baseline

| Appraiser | Decision correctness | Within-appraiser same-label match | Both trials correct on sample | Missed-NG / all NG decisions | False rejects / all OK decisions |
| --- | ---: | ---: | ---: | ---: | ---: |
| 1 | 86/100=86% | 46/50=92% | **41/50=82%** | 13/28=46.43% | 1/72=1.39% |
| 2 | 88/100=88% | 50/50=100% | **44/50=88%** | 12/28=42.86% | 0/72=0% |
| 3 | 79/100=79% | 47/50=94% | **38/50=76%** | 17/28=60.71% | 4/72=5.56% |

- **Do not** describe 86/100 per-decision accuracy as “41 of 50 samples correct in all trials”; these are different.
- **Do not** label 46/50 within-appraiser consistency as 46/50 reference correctness.
- **Do not** pass a study based only on 100% repeat consistency; appraiser 2 misses 42.86% of reference NG decisions.
- Kappa exactly 0.4 / 0.75 compared on full precision; reject spreadsheet's malformed inequality.
- Pe=1 ⇒ Kappa Not Calculable; zero denominator for NG/OK conditional error rates ⇒ Not Calculable.
- Unbalanced classes require explicit sample composition in reports; do not invent a confidence interval assuming 100 distinct independent parts.
- Any attempt to compute an overall pass across all appraisers requires a separate approved aggregation/decision policy.

## 4. QA exit criteria

1. Reference input fixtures approved by domain reviewer, stored with appropriate confidentiality.
2. Mathematical implementation and an independent reference validator agree within specified precision tolerances.
3. All error and threshold cases tested.
4. Method and calculation version recorded with each result and immutable finalized revision.
5. PDF labels denominators and methods explicitly; no stale results after edits.
6. Ratama/customer signs off on Kappa aggregation choice, missed-NG risk policy, %Tolerance and final combined verdict.

**No freeze or release is authorized from this matrix alone.**
