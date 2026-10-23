# Attribute MSA — Study Input Workflow v0.1

**Status:** DRAFT — proposed UX, NOT FROZEN  
**Review date:** 2026-10-23

## Business scenario

Ratama's customer may collect Attribute MSA results on **one combined paper sheet** or on **one sheet per appraiser**. Both must lead to **one MSA study** without duplicate records or mixing appraiser identities.

The initial supported example is **50 distinct samples × 3 appraisers × 2 trials = 300 OK/NG appraiser decisions** plus **one known reference per sample**.

## Recommended MVP: one study, one authorized data-entry owner

1. Authorized **Operator or Supervisor creates one Attribute study** and enters study metadata.
2. Study setup lists three appraisers (the actual people who inspected parts). These are **domain identities**, not necessarily application-login accounts.
3. System displays **one input section/tab per appraiser** (A, B, C). Each contains 50 sample IDs × Trial 1 and Trial 2. Design supports keyboard navigation and paste from Excel.
4. An authorized person enters each appraiser's original recorded decisions, regardless of whether the source was one collective sheet or three separate sheets. All input remains under the same **study ID**, with no new study created per paper.
5. Reference OK/NG values are entered/validated in a **separate restricted section** (normally by Supervisor or authorized quality staff); do **not** expose reference labels to people still conducting blind inspections.
6. Draft can be saved with incomplete input, with per-appraiser completion progress. **Calculate is blocked** until all 300 decisions and 50 references are present and valid.
7. Calculate outputs **individual appraiser Kappa and agreement/error indicators**, plus an overarching **Supervisor Review** section. Do not silently average three Kappas to manufacture an overall passing number.
8. Authorized user finalizes only after result review; prior calculations become stale if any source value is edited, and finalized revisions remain immutable.

## Permissions and attribution

- Preserve previously agreed model: **Operator can edit their own studies**, **Supervisor can edit/manage studies owned by others**, **Admin manages accounts**.
- **Recording account (`entered_by`)** and **measurement appraiser (`appraiser_id`)** are independent concepts. Track changes per user/account and per data slot.
- In the first MVP, only the **study owner** or **Supervisor** edits the study (including all three appraiser sections). This supports both paper collection patterns without new access permissions.
- An Operator cannot directly edit a separate owner's study or another appraiser's section under that owner's study.
- Future optional **assigned contributor per appraiser section** may allow multiple users to enter data into the **same study**. That requires explicit per-section authorization, revision/conflict handling, and an approved exception to the current owner-edit policy. **Not included in the initial MVP.**

## Input example and expected behavior

| Physical collection | Digital input |
| --- | --- |
| One paper listing all 3 inspectors | Authorized study owner fills A, B, C tabs one after another |
| Three separate paper sheets | Authorized study owner fills A, B, C tabs independently; can save draft between tabs |
| One appraiser sheet arrives later | Keep study in Draft; do not calculate as if that person's data were zero |
| Another operator tries to edit the owner's study | Access denied, unless the account is Supervisor |
| Reference not verified | Do not finalize the Kappa calculation |

## Study outputs

Per-appraiser:
- Pooled Kappa vs Known Reference, source-based Ratama judgment label.
- Percent decision-level correct agreement, repeated-label consistency (trial 1 = trial 2), percent samples correct in **both** trials, missed NG and false reject with clear denominators.
- Avoid implying repeat consistency equals measurement correctness.

Whole-study summary:
- Show status/counts for each appraiser **separately**.
- Proposed default **Study Review Required** if any appraiser is Improvement/Poor or if a significant defect-detection risk is visible; the final customer-specific severity policy needs Ratama validation.
- No independently invented mean Kappa, pooled-across-appraiser single-score threshold, or globally certified OK decision.

## Design notes for validated study collection

- Plan a balanced Attribute study where feasible, using representative OK/NG samples with known references and randomized/repeated blind classifications.
- Minitab's published guidance recommends at least 50 Attribute samples, at least two ratings per sample/appraiser, and ideally at least three representative appraisers. These are design guidelines, not universal requirements overriding customer approval.
- Avoid revealing the reference answer before an appraiser has finished independent classifications.
- If the reference cannot be established reliably, disable Kappa vs Reference rather than treating unknown as OK/NG.

## Open for further design review

- Exactly which app roles may input/approve the reference.
- Whether a Supervisor must review before all Finalize actions.
- Whether parallel per-appraiser entry is critical for the first customer (deferred by default).
- How paper records are traced or retained in the client's quality-management process.

Reference:
- https://support.minitab.com/en-us/minitab/help-and-how-to/quality-and-process-improvement/measurement-system-analysis/how-to/attribute-agreement-analysis/attribute-agreement-analysis/before-you-start/data-considerations/
