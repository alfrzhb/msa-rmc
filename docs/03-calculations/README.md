# MSA Calculation Specifications

**Status:** DRAFT — AUDITED, NOT FROZEN (2026-10-22)

This folder documents the independent review of Ratama's Excel examples and training methods.

## Read in order

1. [Ratama Source Audit v0.1](ratama-source-audit-v0.1.md) — source inventory, study-design differences, independently recalculated outputs, errors and risks.
2. [Variable MSA Calculation Specification v0.1](variable-msa-calculation-spec-v0.1.md) — Average & Range draft, Ratama preset, formulas, tolerances, NDC and edge cases.
3. [Attribute MSA Audit v0.1](attribute-msa-audit-v0.1.md) — Kappa-vs-Reference audit, agreement, missed-NG and false-reject indicators.

## Decision state

- **Agreed direction:** Variable Average & Range with 10 × 3 × 3 Ratama preset, paired %Study Var/%Tolerance reporting, NDC and proper numerical corrections.
- **Proposed but not approved:** Final judgment composite, Attribute Kappa method and companion safety metrics, threshold boundary policy, alternative study sizes.
- **Unverified implementation:** Raw XLSX formula-cell dependency scan and runnable application tests.

Existing workbook results that contradict verified formulas are *not authoritative*. Original source workbooks have not been copied to this public repository.
