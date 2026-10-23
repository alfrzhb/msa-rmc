# MSA Calculation Decisions v0.3 — Review Acknowledgment

**Status:** DRAFT / DECISIONS ACCEPTED FOR DESIGN, NOT FROZEN  
**Date:** 2026-10-23  
**Context:** User accepted decision items 1, 2, 4 and 6 from the six-gate Final Calculation Review, recommended standards-based handling for item 5, and asked for a usable Attribute study data-entry workflow for item 3.

## Review outcome

| Previous gate | User direction | MVP policy proposed/accepted for design | Still open |
| --- | --- | --- | --- |
| G1 — Attribute Kappa | “Ikut saja” (accept recommendation) | **Ratama Pooled Kappa vs Known Reference** per appraiser is versioned primary output; not equivalent to Minitab trial-average Kappa | Ratama domain signoff that this is the method to publish |
| G2 — Variable judgement | Accept recommendation | Use Ratama %Study Var bands; show NDC and R-chart checks separately; conflicting signals ⇒ **Review Required**; display %Tolerance as distinct metric, never silently combine it into a new numeric threshold | Domain approval of escalation and tolerance's final role |
| G3 — Attribute judgement and data entry | User requests a practical recommendation due mixed paper collection | **One study / three appraiser sections**, single authorized owner/Supervisor entering all source decisions for MVP; each appraiser has individual Kappa and error indicators, no automatic global mean Kappa pass | Customer acceptance for missed NG/false reject; who finalizes; whether collaborative entry is later required |
| G4 — Thresholds and number formatting | Accept recommendation | Variable GRR <10% OK; 10–30% conditional; >30% NG. NDC≥5 candidate. Attribute pooled Kappa≥0.75 OK, [0.40, 0.75) Improvement, <0.40 Poor; compare on full numeric precision, round only for display. For positive GRR use **literal Ratama floor(NDCraw)**, including NDC=0 if raw<1; identify convention as Ratama-specific and do not claim exact Minitab parity | Explicit Ratama domain confirmation of the exact boundary interpretations and low-end NDC convention |
| G5 — Study validity | Follow applicable standards/guidance | Crossed Variable design, representative selection, randomized measurements, actual trained appraisers, calibrated equipment, known and blinded Attribute references, balanced repetitions. Minitab guidance and Ratama PPT are operational references; IATF 16949 acceptance/customer alternative rules apply **when contract/sector requires them** | Actual client quality-system and customer-specific acceptance evidence |
| G6 — QA | Accept recommendation | Golden datasets, independently recomputed corrected values, formula version, precision cases, limits, negative AV², zero variation, Pe=1, study revision and reporting tests; retain historical calculations per finalized revision | Regression suite not yet implemented or executed; formal acceptance fixtures still need review |

## Variable overall-result policy for MVP

1. Show *calculated* %GRR Study Variation band (Ratama 10% and 30% thresholds) separately from other factors.
2. Show *measured* NDC>=5 pass/fail (when calculable).
3. Show R-chart subgroup control warnings and %Tolerance when valid paired LSL/USL exist.
4. A Conditional band, failed NDC, or a significant R-chart alarm triggers **Review Required** for final disposition, without overwriting the underlying GRR band.
5. An NG band remains NG even if NDC passes; *Review Required* is a workflow flag, not an AIAG statistical category.
6. %Tolerance is informational/fitness-for-purpose; no additional global pass/fail threshold is invented until Ratama/client states the use case and acceptance policy.
7. If results are undefined (e.g. zero denominator), show **Not Calculable** and explanatory flags; do not automatically pass.

## Attribute per-appraiser assessment and combined study view

- Pooled Kappa calculated per appraiser from all trials and the known reference; store method name/version.
- Show per-appraiser Kappa classification, per-decision accuracy, trial consistency, both-trials-correct sample agreement, missed NG, false reject.
- A study contains exactly one sample/reference set and separate appraiser input sections; appraiser can be different from the logged-in data-entry user.
- The aggregate study page is a review panel showing **individual findings**; not a new pooled/average global Kappa result.
- A global study quality verdict and numeric limits on missed NG and false reject remain **OPEN** for Ratama/client review.

## Input and sampling guidelines

1. Variable: same identifiable parts measured by all selected appraisers in a crossed, repeatable design; representative range, random order, minimum repeated trials and appropriately calibrated instrument.
2. Attribute: known sample reference, sufficient and representative balance of OK/NG, random order, same number of repetitions by each appraiser; do not expose reference during independent inspection.
3. Initial workbook presets are supported **examples**, not unconditional proof of statistical adequacy.
4. Do not label Kappa pooled over repeated judgments as though all 100 decisions were 100 independent physical samples.
5. If an alternative sample count or method is introduced, revalidate constants/formulas and keep explicit method version.
6. The user's request “ikut standar” does **not** override customer-specific MSA requirements or grant formal quality-system approval.

## Source citations and provenance

- Ratama's `Form Analisa MSA Variabel_MDI.xlsx`, `Form MSA ATTRIBUTE KAPPA_PT XYZ.xlsx`, `Modul Coretools_MSA_220930_TVS.pptx` (prior audit retained in this repo; raw customer files not republished).
- Minitab Crossed Gage R&R data considerations: https://support.minitab.com/en-us/minitab/help-and-how-to/quality-and-process-improvement/measurement-system-analysis/how-to/gage-study/crossed-gage-r-r-study/before-you-start/data-considerations/
- Minitab Attribute Agreement data considerations: https://support.minitab.com/en-us/minitab/help-and-how-to/quality-and-process-improvement/measurement-system-analysis/how-to/attribute-agreement-analysis/attribute-agreement-analysis/before-you-start/data-considerations/
- IATF 16949 (2016) §7.1.5.1.1: statistical methods and acceptance criteria conform to MSA reference manuals; other approaches permitted subject to customer approval where applicable.

**No FROZEN designation, production claim, or code implementation is authorized by this document.**
