# RMC MSA — High-Fidelity UI Design Review v0.1

**Status:** PROPOSED VISUAL DESIGN / REVIEW DRAFT — NOT FROZEN  
**Date:** 28 October 2026  
**Based on:** [Approved Responsive Wireframe v0.2](wireframe-responsive-review-v0.2.md), [Field & Screen Specification v0.3](field-and-screen-specification-v0.3.md) and DEC-027 through DEC-031.

## Recommendation

Adopt **Industrial Workspace**, a professional engineering product language, rather than a visually busy consumer SaaS dashboard. The visual design must increase readability and input accuracy and must not change MSA methodological assumptions.

### Design tokens

| Purpose | Token | Use |
| --- | --- | --- |
| Deep Navy | `#102733` | Desktop sidebar and stable information hierarchy |
| Teal | `#0C897E` | Primary buttons, active page, keyboard focus |
| Teal tint | `#E9F7F4` | Active wizard step and selected content |
| Canvas | `#F4F7F9` | Workspace background |
| Surface | `#FFFFFF` | Forms, data tables, reports and cards |
| Ink | `#172B38` | Primary text |
| Secondary text | `#647887` | Metadata and supporting descriptions |
| Border | `#DBE5EB` | Subtle section separation |
| Amber | Semantic | Pending Supervisor review and Review Required |
| Green | Semantic | Finalized workflow status; never substitute for missing statistical policy |

System font stack: Inter / Segoe UI / system; no external assets or company logo were manufactured.

## Responsive components

- **Desktop:** dark persistent sidebar, compact top bar, full Excel-like grid for 90 Variable readings or 300 Attribute classifications, clearly labeled appraiser tabs.
- **Tablet:** cards reflow, condensed global navigation, major workflows remain accessible.
- **Mobile:** compact sticky top bar and bottom navigation, all-study History cards; Variable input per part (three numeric trials); Attribute input per sample (two OK/NG trials) with jump and Previous/Next.
- **Attribute result redesign:** mobile cards per appraiser show pooled Kappa, Kappa band, Missed NG (numerator/denominator and percentage), and False Reject (numerator/denominator and percentage) **without hidden sideways-scrolling columns**.
- **Results:** calculated metric band distinct from policy-approved overall disposition; warning and Quality approval status precede expandable technical formulas.
- **Roles:** all Operators may read History but edit only own Drafts; Supervisor alone maintains Known Reference and Approves & Finalizes.
- **PDF:** placeholder for corporate logo and authorized approval signature; no fake asset or claim of certified signature.

## Prototype delivery

A standalone high-fidelity HTML preview, screenshots, one overview PNG, QA results and a ZIP package were delivered in the conversation. These files are **not the production application** and are not included as code in this documentation-only GitHub repository.

All values are demonstration examples. The HTML does not perform real MSA calculations, persistent saves, authentication, authorization, Supervisor approvals or signed-PDF issuance.

## Visual prototype QA

- 8 widths: **320, 360, 390, 430, 768, 820, 1024 and 1440 px**
- 3 visual roles: Operator, Supervisor, Admin
- 15 page states per role/width: **360 render checks**
- **0 document-level horizontal overflow; 0 JavaScript page errors**
- Additional simulated flow checks: mobile part movement and input persistence, History search, role switching, Supervisor-only reference display, and prohibition on demo approval under unapproved Quality criteria.

These are **prototype smoke checks**, not production QA, actual role-permission validation, accessibility certification or calculation-regression results.

## Approval gates

- [ ] Product owner reviews and approves visual identity, typography, input usability and mobile risk cards
- [ ] Ratama provides real logo, signature and signatory information
- [ ] Quality approves all customer-specific risk/disposition policies independently
- [ ] Implement a real accessible frontend, backend authorization, calculation regression suite and secure approval/PDF workflow only after technical planning

**No new design approval is assumed by creating this draft.** The last approved UI/UX baseline remains Responsive Wireframe v0.2, DEC-031. Entire requirements and statistical criteria are NOT FROZEN.
