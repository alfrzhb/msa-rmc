# RMC MSA — High-Fidelity UI Design Review v0.1

**Status:** CONCEPT ACCEPTED WITH VISUAL REVISIONS REQUIRED — DRAFT / NOT FROZEN  
**Date:** 28 October 2026  
**Based on:** [Approved Responsive Wireframe v0.2](wireframe-responsive-review-v0.2.md), [Field & Screen Specification v0.3](field-and-screen-specification-v0.3.md) and DEC-027 through DEC-031.

## Product-owner review — 28 October 2026

**Decision:** The product owner considers the overall high-fidelity layout and UX **sufficiently acceptable as a working direction**, with one explicit objection: the current navy/teal colors feel too much like a **generic AI-generated SaaS dashboard** ("AI slop"). This is **conditional acceptance of the design concept**, **not approval of the current color palette or final visual identity**.

**Keep:**
- Approved responsive information architecture (desktop tables, mobile per-part/per-sample input, History cards).
- Usability-focused hierarchy, MSA results-first view, permissions, and Supervisor approval layout.
- Minimal distraction and readable data presentation.

**Required visual polish before final frontend styling:**
1. Replace the saturated navy/teal-heavy appearance with a more restrained **enterprise/industrial engineering** character.
2. Reduce unnecessary colored surfaces, glowing highlights, gradients and decorative shadows; favor subtle borders, white space and typographic hierarchy.
3. Use color chiefly for **functional meaning** (primary action, focus, Approved, Pending, Warning, Error); do not confuse lifecycle approval with statistically acceptable MSA results.
4. Review contrast, keyboard focus and status accessibility after changing colors; preserve desktop and mobile usability.
5. Present the visual refresh for review as **v0.2** before considering visual design FROZEN.

**Candidate visual tokens (suggested, NOT approved):**

| Role | Proposed color | Use |
| --- | --- | --- |
| Canvas | `#F8F9FA` | Light neutral workspace |
| Sidebar | `#24292F` | Charcoal, replacing blue/navy sidebar |
| Primary | `#456A81` | Muted steel blue, replacing prominent teal |
| Surface | `#FFFFFF` | Clean cards, grids, report |
| Borders | Light neutral gray | Minimal separators, no heavy shadow |
| Fonts | Inter or IBM Plex Sans | Readable enterprise UI |
| Semantic statuses | Color on demand | Approval, pending, warnings and errors only |

**Scope:** Documentation of stakeholder feedback; no HTML/CSS prototype redesign has been applied in this commit. The table of current v0.1 tokens below is retained **only as the as-reviewed prototype's historical baseline**.

## Recommendation

Adopt **Industrial Workspace**, a professional engineering product language, rather than a visually busy consumer SaaS dashboard. The visual design must increase readability and input accuracy and must not change MSA methodological assumptions.

### Design tokens of the reviewed v0.1 prototype (not final)

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

- [x] Product owner provisionally accepts overall high-fidelity concept, layout and usability direction (with color criticism recorded)
- [ ] Product owner reviews and approves revised color palette and final visual identity (visual polish v0.2)
- [ ] Ratama provides real logo, signature and signatory information
- [ ] Quality approves all customer-specific risk/disposition policies independently
- [ ] Implement a real accessible frontend, backend authorization, calculation regression suite and secure approval/PDF workflow only after technical planning

**Current review status:** High-fidelity v0.1 **conditionally accepted as a concept only**; its visual palette is **not final**. The previously approved responsive UX baseline remains Wireframe v0.2 (DEC-031). Entire requirements and statistical criteria are NOT FROZEN.
