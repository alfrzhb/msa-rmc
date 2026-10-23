# MSA Calculation Specifications

**Status:** REVIEW DRAFT — NOT FROZEN (2026-10-22)

## Current review package

**Start here:** [Calculation Decisions v0.3](calculation-decision-review-v0.3.md), then [Final Calculation Review v0.2](final-calculation-review-v0.2.md)

The review reconciles Ratama's intended methods and the independently recomputed Variable/Attribute workbook data. It also distinguishes **Ratama pooled Kappa** from **Minitab mean per-trial Kappa**, and per-decision agreement from per-sample agreement. It documents final approval gates; it does not freeze the project.

## Supporting audit trail

1. [Ratama Source Audit v0.1](ratama-source-audit-v0.1.md) — source inventory, underlying study protocols, source anomalies and numerical risks.
2. [Variable MSA Calculation Specification v0.1](variable-msa-calculation-spec-v0.1.md) — Ratama Average & Range, 10×3×3 preset, corrected formulas, NDC and tolerance.
3. [Attribute MSA Audit v0.1](attribute-msa-audit-v0.1.md) — Ratama 50×3×2 pooled Kappa and decision-level class-conditional risk indicators.
4. [Calculation Review Regression Matrix v0.2](../06-quality-assurance/calculation-review-test-matrix-v0.2.md) — additional method-comparison and counting-policy tests.

## Current position

- **Direction accepted:** Variable Average & Range; Variable 10×3×3 preset; Attribute Kappa-vs-reference with agreement/error diagnostics candidate.
- **Verified:** Arithmetic from 90 Variable measurements and 300 Attribute decisions, including correction of Excel defects.
- **Open:** Ratama final Kappa method signoff, final composite decision rules, threshold boundaries, low-NDC convention, error-risk policy and executable implementation tests.
- **Do not claim:** Exact Minitab output parity for pooled Kappa, or FROZEN approval.

## Product decision update — 2026-10-23

The user accepted G1/G2/G4/G6 product design recommendations and asked G5 to follow recognized MSA study standards. Ratama/customer domain approval and executable calculation regression tests remain pending. Attribute paper-to-digital workflow is documented separately in [Attribute Study Input Workflow v0.1](../04-ui-ux/attribute-study-input-flow-v0.1.md).
