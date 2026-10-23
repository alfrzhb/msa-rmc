# RMC MSA — Wireframe Review v0.1

**Status:** LOW-FIDELITY / DRAFT — NOT FROZEN  
**Prepared:** 23 October 2026  
**Based on:** [Field & Screen Specification v0.3](field-and-screen-specification-v0.3.md) and [Attribute Study Input Workflow v0.1](attribute-study-input-flow-v0.1.md)

## Prototype deliverable

A standalone, interactive HTML wireframe and desktop/mobile screenshots were produced for stakeholder review in the ChatGPT conversation (together with a ZIP package). The interactive file and screenshots are **not committed in this repository**; this Markdown file records its design coverage and acceptance checkpoints, not an application release.

**No** production framework, real statistical engine, authentication, database, signatory upload, PDF generator, final approval workflow, or customer confidential data was implemented.

## Design system

- Low-fidelity grayscale with neutral borders, clear typography, desktop-first workspace.
- Left global navigation; persistent study context, method and status.
- Four-step study workflow: Setup → Appraisers & Samples → Input → Results & Review.
- Field visibility: required Core fields first; optional Advanced Details collapsed.
- Three appraiser input tabs, keyboard/paste-oriented grids.
- Results hierarchy: source statistical band versus approved overall disposition, warning and next step, key metrics, expandable Calculation Details.
- Separate Supervisor-only Known Reference management; no reference leakage to Operator during blinded data collection.
- Operator can see all History but edit only own Draft; Supervisor alone Approves & Finalizes.
- Official report wireframe contains **company logo placeholder** and **authorized Supervisor signature placeholder**. No fabricated sign-off or claim that an image equals certified digital signature.

## Screen coverage

| Screen ID | Wireframe | Critical interaction |
| --- | --- | --- |
| S01 | Login | Employee-only sign-in, no public registration |
| S02 | Dashboard | Studies, Drafts, approval queue and quick actions |
| S03 | MSA History | Search/list all; owner-write versus other-owner read-only |
| S04 | New MSA | Variable and Attribute selection |
| S05 | Study Setup | Core fields, specifications, optional Advanced Details |
| S06 | Appraisers & Samples | Three distinct appraisers and sample IDs |
| S07 | Variable Input | 10 parts × 3 trials × 3 appraisers, editable numeric grid |
| S08 | Attribute Input | 50 samples × 2 trials × 3 appraisers, separate restricted Known Reference |
| S09 | Results & Review | Judgement/warnings first, expandable methods, Submit for Approval |
| S10 | Study Detail | Status, revisions and approval provenance |
| S11 | Report PDF | Logo and Supervisor approval/signature placeholders |
| S12 | User Management | Admin account management |
| S13 | Judgment Policy | Supervisor view of approved policy and pending criteria |

Supervisor Approval Queue is a supporting screen for the S09/S10 review workflow, not a second approval tier.

## Validation performed on prototype (not the future application)

- 12 desktop state screenshots plus one responsive mobile dashboard captured using a browser.
- Interactive flows checked: Create → Setup → Appraisers → Variable grid → Results → Submit (simulation only).
- Role simulations checked: Operator reference access denied; Supervisor may view/verify reference; Admin is directed to User Management.
- Read-only other-owner Draft behavior confirmed in the prototype.
- Supervisor Approve & Finalize is **blocked by design** under the unapproved sample risk policy.
- Browser check reported no JavaScript page errors; mobile Dashboard does not overflow horizontally.
- **These checks validate only prototype navigation/rendering**, not security, mathematical validity, policy compliance or application functionality.

## Stakeholder UX checklist

- [ ] Four-step flow is usable with minimum training
- [ ] Study Setup asks the right Core fields without excessive form complexity
- [ ] Variable and Attribute grids are convenient for real paper/Excel input
- [ ] Known Reference is fully separated from appraiser classification
- [ ] Operator sees all histories without gaining edit or blind reference privileges
- [ ] Warnings and approval rules are readable without opening formulas
- [ ] Supervisor can review, request correction and approve only when policy allows
- [ ] PDF layout is acceptable once genuine logo/signature assets are available

## Before moving to high fidelity

Review workflow with Ratama Operator and Supervisor. Confirm branded PDF layout, signatory/asset requirements, exact validation errors for paste-from-Excel, reporting appendix and customer Quality acceptance rules. Reconcile changes back into [Field & Screen v0.3](field-and-screen-specification-v0.3.md) with a future revision; do not mark any document FROZEN solely because a wireframe exists.
