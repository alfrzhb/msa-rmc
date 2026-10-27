# RMC MSA — Responsive Wireframe Review v0.2

**Status:** PRODUCT-OWNER UX DIRECTION APPROVED (28 October 2026); ARTIFACT STILL DRAFT / NOT FROZEN  
**Date:** 28 October 2026  
**Based on:** [Field & Screen Specification v0.3](field-and-screen-specification-v0.3.md), [Wireframe Review v0.1](wireframe-review-v0.1.md), DEC-027..030.

## Acceptance decision

On **28 October 2026**, the product owner explicitly accepted Responsive Wireframe v0.2 as the **UI/UX design direction**. This sign-off covers responsive layouts and intended interaction patterns, not approval of statistical methods, production readiness, real signatures, live role enforcement or the complete specification. See [DEC-031](../decisions/decision-log.md).

## Intent

Make the low-fidelity interactive HTML wireframe usable on mobile, tablet and desktop, without changing statistical or approval semantics. Standalone HTML, preview screenshots and a ZIP archive were delivered in conversation. This is not a production application.

## Responsive UX decisions

| Area | Desktop | Mobile |
| --- | --- | --- |
| Navigation | Persistent sidebar and breadcrumb | Compact sticky top bar, bottom nav, role-aware More menu |
| Dashboard | KPI tiles, recent-study table | Reflowed KPI tiles, cards, prominent primary action |
| History | Search and full-width study table | Individual study cards without horizontal scrolling |
| Setup | Two-column Core form, Advanced Details expandable | Touch-friendly single-column form |
| Wizard | Full four-step labels | Condensed four-step progress |
| Variable | Ten-part by three-trial matrix per appraiser | One part per view, three inputs, jump and previous/next |
| Attribute | Fifty-sample by two-trial matrix per appraiser | One sample per view, two OK/NG controls, jump and previous/next |
| Known Reference | Supervisor-only reference table | Supervisor-only focused reference editor |
| Results | Judgement, warnings and key metrics, details expandable | Priority order preserved in stacked layout |
| Report | Printable wide report | Responsive document preview and genuine-asset placeholders |

**Role model unchanged:** Operators can view all History but edit only their own Draft; Supervisor alone manages Known Reference and Approves & Finalizes. Visual role switching in the prototype is not actual security.

## QA of prototype layout/interactions

- Viewports: 320, 360, 390, 430, 768, 820, 1024, and 1440 pixels.
- For each viewport: 3 role presentations times 15 page states.
- Browser layout check: zero global horizontal overflow, zero JavaScript page errors.
- Interaction checks: mobile sample jump and next/previous; numeric editing; appraiser tabs; More menu; study cards; Operator reference denial; Supervisor reference access; blocked approval when risk policy is unapproved.
- Desktop full input matrix preserved; mobile uses one part/sample per view.
- All checks apply to **prototype only**, not backend security, actual calculations, accessibility audit, approval processing or PDF issuance.

## Review artifacts

The HTML prototype, responsive overview PNG, and ZIP of selected screenshots were supplied as conversation files. They are intentionally **not committed as product source code** in the documentation-only repo. The prior [wireframe review v0.1](wireframe-review-v0.1.md) is preserved.

## Pending Ratama review

- Try mobile data entry with actual operators and representative paper/Excel workflows.
- Verify desktop entry efficiency for 90 Variable readings and 300 Attribute decisions.
- Verify all-history viewing is clearly read-only for other people's Drafts.
- Approve final logo/signature assets and the report's printable layout.
- Ratama Quality must separately approve customer-specific acceptance criteria and exceptional dispositions.

**Product-owner UI/UX direction accepted; documentation remains DRAFT / NOT FROZEN. No production code or MSA formula was changed.**
