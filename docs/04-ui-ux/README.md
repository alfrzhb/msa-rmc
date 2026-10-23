# UI / UX Design

**Current:** [Field & Screen Specification v0.3](field-and-screen-specification-v0.3.md) — **DRAFT, NOT FROZEN** (2026-10-23)

## Specifications
- [Field & Screen Specification v0.3](field-and-screen-specification-v0.3.md) — 13 screens, 35 UI acceptance cases, user-confirmed Supervisor approval/finalization, exclusive Supervisor Known Reference management, shared read-only Operator History, official logo/signature PDF.
- [Field & Screen Specification v0.2](field-and-screen-specification-v0.2.md) — preceding UX revision, retained for audit.
- [Field & Screen Specification v0.1](field-and-screen-specification-v0.1.md) — original draft, retained for audit.
- [Attribute Study Input Workflow v0.1](attribute-study-input-flow-v0.1.md) — collection model: one study / three appraiser sections. **Role handling for reference and historical visibility is superseded by v0.3.**

## MVP navigation

Login → Dashboard/All History → Create Study → Setup → Appraisers & Samples → Variable/Attribute Input → Calculate → **Submit for Approval** → **Supervisor Approve & Finalize** → Final PDF.

## Accepted product decisions

- Only Supervisor approves and finalizes studies.
- Only Supervisor creates/edits/verifies Attribute Known Reference (hidden during blind assessment).
- Operators can view every study's History but edit only their own Draft; other finalized reports are read-only.
- Final PDF contains corporate logo and Supervisor signature/approval record (assets supplied later).

**Still pending:** Ratama-approved Quality risk thresholds and exception handling; actual logo/signature assets, specific signatory/title, report layout and any regulated e-signature requirements; UAT. No application code or screen tests yet.
