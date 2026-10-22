# Calculation QA & Regression Plan v0.1

**Status: DRAFT — NOT FROZEN**  
**Audit date:** 2026-10-22  
**Scope:** Statistical correctness and workflow safeguards for RMC MSA MVP.

## 1. Test policy

- Test the **selected mathematical method**, never the raw erroneous spreadsheet cell values.
- Compare against independent numerical calculations and an accepted reference implementation where applicable.
- Use full internal precision; round only when formatting display or reports.
- Preserve reference test fixtures only in an authorized location, not in this public repository until client confirms their release.
- Any method, multiplier, constant, rounding, or judgment change increments **calculation method version** and re-runs the regression suite.

## 2. Variable MVP — 10 × 3 × 3 golden values

Source: Ratama Variable MSA workbook, independently recalculated from all 90 numeric entries.

| Check | Expected | Tolerance / special rule |
| --- | --- | --- |
| Correct data points | 90 | Exact count |
| Rbar | 0.004 | approximately 1e-10 |
| XDIFF | approximately 0 | Ignore floating-point residue; no unwarranted change in branch |
| EV | 0.0023632 | approximately 1e-10 |
| AV | 0 | Exact branch after negative variance correction |
| GRR | 0.0023632 | approximately 1e-10 |
| PV | 0.0241193333333 | approximately 1e-9 |
| TV | 0.0242348293719 | approximately 1e-9 |
| %Study Var | 9.7512549552 | approximately 1e-7 percentage points |
| %Tolerance | 7.0896 | approximately 1e-7 percentage points |
| NDC raw | 14.3907667569 | approximately 1e-7 |
| NDC integer | 14 | Exact, truncation |
| UCL (D4=2.58) | 0.01032 | approximately 1e-10 |
| R-chart above-UCL points | 0 | Exact |

Regression failures must flag workbook-derived **9.910873% GRR** and **39.99688 UCL** as incorrect.

## 3. Attribute MVP — 50 × 3 × 2 golden values

Source: Ratama Attribute workbook. Three appraisers, 300 classification decisions, 50 distinct samples, 14 NG / 36 OK in the reference.

| Appraiser | A | B | C | D | Kappa | Within-match | Missed-NG rate | False-reject rate |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| 1 | 15 | 1 | 13 | 71 | 0.6004566210 | 46/50 | 13/28 | 1/72 |
| 2 | 16 | 0 | 12 | 72 | 0.6575342466 | 50/50 | 12/28 | 0/72 |
| 3 | 11 | 4 | 17 | 68 | 0.3930635838 | 47/50 | 17/28 | 4/72 |

Checks:
- Every confusion row sums to **N=100**.
- Kappa computed from **observed Po and marginal-based Pe**.
- The source workbook's appraiser 3 value **0.4047619** must **not** be accepted as the corrected Kappa.
- For source-based candidate bands, appraiser 3 is below **0.40** and labels as **POOR**.
- Overall agreement and within-appraiser repeatability are separate metrics; appraiser 2 demonstrates 100% within-match but has missed-NG rate 42.86%.
- Reference categories must preserve the actual 14/36 split, not a fabricated balanced dataset.

## 4. Boundary / bad-input tests

| Test | Expected behavior |
| --- | --- |
| Missing numeric measurement | Study may save Draft; calculation blocked with row/column identification |
| Non-finite numeric measurement | Rejected; do not show NaN/Infinity |
| Duplicate entry in part-appraiser-trial slot | Rejected or deliberately updated via authorized edit; never counted twice |
| AV² candidate < 0 | AV=0, warning/event trace captured |
| GRR=0 | No divide-by-zero; NDC=Not Calculable (reason shown) |
| TV=0 | No divide-by-zero; %Study Var=Not Calculable |
| USL<=LSL, missing tolerance, invalid units | %Tolerance not calculated; descriptive result retained |
| %Study Var exactly 10% or 30% | Evaluated by full precision with agreed boundary conditions |
| NDC raw just below / just above 5 | Integer truncation applied; threshold evaluated on integer NDC |
| R-range above UCL | Independent warning and investigation guidance; not hidden by favorable %GRR |
| Unrecognized Attribute category | Rejected; never silently mapped to OK |
| Missing Attribute reference | Kappa-vs-reference calculation blocked |
| Pe=1 | Kappa=Not Calculable, not 1 or 0 by default |
| No NG reference judgments | Missed-NG rate=Not Calculable (denominator zero) |
| No OK reference judgments | False-reject rate=Not Calculable |
| Kappa exactly 0.40 or 0.75 | Compare at full precision to explicitly approved thresholds |
| Edited input after Calculate | Prior results flagged stale until recomputed |
| Finalized revision | Original finalized measurements/results/PDF remain unchanged |
| Operator edits another operator's study | Denied by server authorization |
| Supervisor revises study | Allowed only via properly audited revision; original kept |

## 5. Review gates before implementation freeze

1. Confirm crossed Variable study assumptions and permitted sizes.
2. Confirm six-sigma tolerance factor and final judgment policy with Ratama/client.
3. Confirm Attribute primary Kappa/secondary indicators and customer-specific missed-NG policy.
4. Approve calculation constants / version / golden fixtures.
5. Inspect the underlying XLSX formulas and workbook dependency chains if formula-cell-level audit is required.
6. Run independent executable reference tests after implementation (not executed at the present documentation stage).

**Current state:** Audit calculations were performed during requirements discovery; automated tests in the application repository have **not** been written or run.
