# Attribute MSA — Kappa & Agreement Audit v0.1

**Status:** DRAFT / METHOD DISCUSSION OPEN — NOT FROZEN  
**Date:** 2026-10-22  
**Scope:** Recompute Ratama Attribute example and identify safe MVP analysis boundaries.

## 1. Study and source data

Primary workbook: **Form MSA ATTRIBUTE KAPPA_PT XYZ.xlsx**. Input structure:
- 50 distinct samples.
- 3 appraisers.
- 2 trials per sample per appraiser.
- Binary **OK / NG** classifications.
- Known, one-per-sample reference.
- 50 × 3 × 2 = **300 actual appraiser decisions**.
- Reference composition: **36 OK**, **14 NG** distinct samples.

The Ratama PPT includes a separate example describing **20–30 parts**; this is not the same as the 50-sample workbook. Do not encode 50 as a universal MSA requirement.

## 2. Workbook-compatible, independently corrected statistic

The source workbook's intended primary analysis is **each appraiser's pooled trial decisions versus the known reference**.

For the 2×2 confusion table:

| | Reference NG | Reference OK |
| --- | ---: | ---: |
| Appraiser NG | A (correct NG) | B (false reject) |
| Appraiser OK | C (missed NG) | D (correct OK) |

For each appraiser across both trials:
- **N** = A+B+C+D = 100 decisions.
- **Po** = (A+D)/N.
- **Pe** = ((A+B)(A+C) + (C+D)(B+D)) / N².
- **Kappa** = (Po−Pe)/(1−Pe), only if 1−Pe > 0.

This is the ordinary two-class chance-corrected Cohen agreement formula, applying Ratama's *pooled per-appraiser-versus-reference* choice. Pairing 2 trials against a reference yields descriptive 100-decision counts per appraiser. The number of **independent distinct samples remains 50**, not 100.

## 3. Recomputed values and workbook errors

| Anonymized appraiser | A/B/C/D | Po | Pe | Correct Kappa | Displayed workbook Kappa |
| --- | --- | ---: | ---: | ---: | ---: |
| 1 | 15 / 1 / 13 / 71 | 0.860000 | 0.649600 | **0.600457** | 0.618096 |
| 2 | 16 / 0 / 12 / 72 | 0.880000 | 0.649600 | **0.657534** | 0.676851 |
| 3 | 11 / 4 / 17 / 68 | 0.790000 | 0.654000 | **0.393064** | 0.404762 |

**Critical:** Appraiser 3 changes from above to below **Kappa = 0.40** after correcting the formula. This can change the reported decision. The calculated crosstab counts are usable, but not the workbook's final Kappa output.

## 4. Interpretation bands from Ratama example

- Kappa ≥ 0.75 → **OK**, as stated by the workbook.
- 0.40 ≤ Kappa < 0.75 → **IMPROVEMENT**.
- Kappa < 0.40 → **POOR**.
- The workbook's printed middle inequality is malformed; use the logically consistent interval above.
- Ratama narrative material discusses “>0.75”; exact treatment of the **boundary at 0.75** needs a formally recorded decision, although the workbook uses ≥0.75.

The labels above are **source-based candidate thresholds**, not a final client-approved universal acceptance rule.

## 5. Agreement, misses and false alarms: recommended explanatory indicators

Computed from *the same* data, without a new input flow:

| Appraiser | %Correct against reference | Within-appraiser match (same sample across 2 trials) | Missed NG rate | False reject rate |
| --- | ---: | ---: | ---: | ---: |
| 1 | 86% | 46/50 = 92% | **13/28 = 46.43%** | 1/72 = 1.39% |
| 2 | 88% | 50/50 = 100% | **12/28 = 42.86%** | 0/72 = 0% |
| 3 | 79% | 47/50 = 94% | **17/28 = 60.71%** | 4/72 = 5.56% |

- **Within-appraiser agreement** = number of samples where Trial 1 matches Trial 2 / 50.
- **Missed NG rate** = count appraiser OK when reference NG / total reference NG decisions = C/(A+C).
- **False reject rate** = count appraiser NG when reference OK / total reference OK decisions = B/(B+D).
- Denominators must be shown; an error count / all observations is **not** the same conditional rate.
- Operator consistency alone does not establish operator accuracy. In this example an appraiser with **100% within-appraiser agreement** still **misses 42.86% of reference-NG decisions**.

**Recommendation for design review:** report Kappa as source-compatible primary result, but display these conditional error rates as warning indicators. Do not invent acceptance thresholds for these indicators without Ratama/customer approval.

## 6. Edge cases and safe computational behavior

- Every sample needs one unambiguous known reference (OK/NG) for *Kappa vs Reference*.
- Each appraiser makes exactly two binary decisions for each sample under this MVP preset; no missing decisions when computing final results.
- Normalize value labels explicitly; reject unknown/free-text statuses rather than silently mapping them.
- If Pe=1, Kappa is **Not Calculable**, even if Po=1. Never show 1.0 by default.
- If there are **zero NG reference judgments**, missed NG rate is **Not Calculable**, not 0%.
- If there are **zero OK reference judgments**, false reject rate is **Not Calculable**, not 0%.
- Use unrounded Kappa for comparisons at 0.4 / 0.75; round only the display.
- Preserve source categories, exact confusion counts, calculation version and results revision.
- The same 50 samples are repeated: treat samples as clustered for future confidence intervals; do not invent independently sampled 100-item CI.
- Study plan should avoid leaking known reference answers to appraisers during blind inspections.

## 7. Pending domain/product decisions

1. Confirm **pooled Kappa vs Reference** as primary MVP method; user explicitly accepted the Variable method but Attribute approval remains open.
2. Confirm thresholds and whether multiple appraisers are aggregated into one overall status.
3. Decide whether the Attribute UI exposes only Kappa, or additionally accuracy, within agreement, missed NG and false reject indicators. The latter are strongly recommended.
4. Determine customer-specific consequences of missed NG judgments and acceptance of any alternative criterion.
5. Validate sample composition and study protocol: reference category balance, randomized order and repeat blinding.
6. Decide whether supervisor can change study sizes later; different designs require revalidated analytics and tests.

**No automatic Attribute release verdict should be based only on a high percentage of agreement.**
