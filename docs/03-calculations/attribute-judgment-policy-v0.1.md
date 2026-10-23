# Attribute MSA — Judgment Policy v0.1

**Status:** DRAFT — PRODUCT DESIGN AGREED, NOT FROZEN  
**Date:** 2026-10-23  
**Decision:** Standard Preset + Configurable Approved Policy for Attribute MSA MVP.

## 1. Principle

- **Formula logic is fixed and versioned, not configurable by end users.** Ratama pooled appraiser Kappa vs Known Reference and its validated companion metrics retain their mathematical definitions. Changes to statistical methods require a reviewed calculation-engine release.
- **Acceptance criteria are policy configuration, not formula configuration.**
- One company/client MVP; do not implement multi-tenant policy management yet.

## 2. Default and configurable policies

- Default policy is **Ratama-aligned**, with candidate per-appraiser Kappa bands: Kappa >= 0.75 = OK; 0.40 <= Kappa < 0.75 = Improvement; Kappa < 0.40 = Poor. Confirm precise boundaries with Ratama/Quality before FROZEN.
- Display missed-NG = C/(A+C), false reject = B/(B+D), accuracy, and repeatability separately with denominators and undefined-case handling.
- **No universal missed-NG or false-reject acceptance limit is assumed.** Ratama/customer Quality must approve exact limits for the client's inspection risk.
- A policy may be configured/selected for a study by Supervisor **only after documented approval** by the appropriate Quality authority. Operators cannot edit criteria.
- Changes must include policy identifier, version, effective time, approval evidence/authority, author, and reason; every change is in the audit trail.
- Draft/unapproved policies may be inspected but **must not** be activated for production acceptance decisions.
- If required acceptance thresholds or overall-study criteria are not approved, show separate measured metrics and mark final overall disposition **Review Required / No Approved Acceptance Policy**, rather than automatically passing.

## 3. Lifecycle and reproducibility

1. Study created with the applicable approved policy version (or no active policy, with explicit pending state).
2. Calculations record method + calculation version, raw inputs, appraisers, metric values and denominators.
3. Evaluation uses the chosen **approved policy version** and full-precision metric values (round only for display).
4. Supervisor reviews exceptions and final disposition; no silent averaging of Kappa across appraisers.
5. At Finalize, retain a **snapshot of policy contents, identifier, version, and result** in the immutable study revision and PDF.
6. Later policy changes apply only according to explicit effective-date and revision rules. They do **not retroactively rewrite** prior finalized results or PDFs.

## 4. Role model (existing agreement remains)

- **Admin:** accounts and role assignment; does not automatically authorize modifying statistical judgments.
- **Supervisor:** selects/configures an approved acceptance policy and manages/reviews studies.
- **Operator:** inputs data into own study, calculates, reads results; **cannot edit formulas or acceptance policy**.
- Quality approval is an **approval authority/evidence**, not a mandatory fourth application role in MVP.

## 5. Pending domain approval

1. Are candidate Ratama Kappa bands acceptable for this client's inspection process?
2. What are the maximum approved **missed-NG** and **false-reject** rates, particularly for critical characteristics?
3. What combination of appraiser-level statuses warrants an overall study OK, Conditional, or NG, versus Supervisor Review Required?
4. Must Supervisor explicitly sign off each finalized Attribute result, or only exceptions?
5. Who provides formal Quality/customer approval evidence, and what is its validity period?

**No calculation formulas, specific risk thresholds, reports, or documents are FROZEN by accepting this policy architecture.**
