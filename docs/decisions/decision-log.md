# Decision Log — RMC MSA Analysis Tool

**Document status:** DRAFT / LIVE REGISTER  
**Last reviewed:** 22 October 2026  
**Basis:** Requirements Analysis v0.3 and stakeholder feedback

This log distinguishes an item accepted in a requirements review from a full, formally `FROZEN` specification.

## Accepted in review (NOT FROZEN)

| ID | Decision | Rationale / impact | Approval scope |
| --- | --- | --- | --- |
| DEC-001 | Start with one client company, defer multi-client management | Keep MVP simple; design for future extensibility | Business direction |
| DEC-002 | Login with individual employee accounts | Traceability and operator accountability | Business direction |
| DEC-003 | Three user roles: Admin, Supervisor, Operator | Separate account administration from quality operations | Reviewed and agreed |
| DEC-004 | Operator can edit own study; Supervisor can manage studies across operators | Controlled editing and supervision | Reviewed and agreed |
| DEC-005 | Manual input and paste-from-Excel for MVP; full XLSX import deferred | Minimize complexity and speed up use | Reviewed and agreed |
| DEC-006 | Finalized studies are not edited in place; changes use a revision | Preserve historical results and traceability | Reviewed and agreed |
| DEC-007 | MSA Variable and Attribute first; SPC later | Focus on validated scope | Business direction |
| DEC-008 | Study history and printable/exportable PDF reports | Core client needs | Business direction |
| DEC-009 | Store evolving specifications as Markdown in GitHub | Reduce risk of losing requirements history | Documentation workflow |\n| DEC-010 | Use Ratama workbooks and training as the functional/reference baseline, but recalculate using validated mathematical formulas | Avoid propagating workbook arithmetic defects or treating the old v1.0 freeze as approval for this rebuilt project | Reviewed and agreed direction |
| DEC-011 | Variable MVP uses Crossed Gage R&R Average & Range | Matches Ratama's Variable workbook; ANOVA and other methods deferred | Reviewed and agreed direction |
| DEC-012 | Use Ratama 10 parts × 3 appraisers × 3 trials as initial Variable preset; Attribute workbook 50 × 3 × 2 is a provisional preset | Keep initial UX simple; training describes other valid sizes, so these are not universal limits | Variable accepted direction; Attribute candidate |
| DEC-013 | Correct erroneous AV and R-chart UCL calculations rather than reproduce spreadsheet displayed errors | Verifiable mathematical correctness is required | Reviewed and agreed direction |
| DEC-014 | Show Variable %Study Var, %Tolerance when specifications exist, NDC and independent range-control information | Ratama training explicitly distinguishes %Study Var and %Tolerance | Reviewed and agreed direction |
| DEC-015 | Record audit findings, method assumptions, golden figures and remaining validation in Markdown; never publish raw client datasets | Traceability and data protection | Documentation workflow |

## Open decisions (DISCUSSION REQUIRED)

| ID | Question | Current proposal |
| --- | --- | --- |
| OPEN-001 | Attribute final MVP method and indicators | Variable Average & Range direction agreed; Attribute Kappa-vs-Reference with safety indicators remains for review |
| OPEN-002 | Flexible vs locked part/appraiser/trial counts | Workbook configurations are initial presets; alternatives need verified constants, protocol and acceptance tests |
| OPEN-003 | Calculation version and final signoff | Source data outputs independently audited; test vectors in calculation documents; raw XLSX formula-cell scan and acceptance pending |
| OPEN-004 | Judgment, threshold boundary and %Tolerance policy | Ratama-based candidate 10/30%, NDC >=5 and Kappa 0.40/0.75; customer confirmation and composite-policy decision required |
| OPEN-005 | Can Operators view all studies or only their own? | Pending |
| OPEN-006 | Exact Admin and Supervisor permissions, including deletion and revision | Proposed matrix needs detailed approval |
| OPEN-007 | Finalize and revision lifecycle details | Proposed, not FROZEN |
| OPEN-008 | Dashboard fields, detailed PDF format, branding | Proposed, not FROZEN |
| OPEN-009 | Technology stack, hosting, backup and security implementation | Decide after requirements and calculations |
| OPEN-010 | Attribute final indicators, missed-NG risk and reference policy | Audited pooled Kappa and large missed-NG rates; customer-specific risk threshold remains open |

## Audit record — 2026-10-22\n\n- [Ratama source reconciliation](../03-calculations/ratama-source-audit-v0.1.md): workbook vs PPT, numerical deviations and interpretation risks.\n- [Variable candidate formula specification](../03-calculations/variable-msa-calculation-spec-v0.1.md): matched Ratama Average & Range input methodology with corrected calculations.\n- [Attribute Kappa audit](../03-calculations/attribute-msa-audit-v0.1.md): corrected reference agreement and missed-NG rates; approval pending.\n- [Regression plan](../06-quality-assurance/calculation-test-plan-v0.1.md): expected values and pending executable tests.\n\n**Decision boundary:** Decisions labeled “agreed direction” do not freeze all formulas, acceptance criteria or method variants. The old Ratama calculation v1.0 FROZEN document is historical evidence only.\n\n## Change control

1. Add new decisions or change proposals to this log.
2. Update the affected Markdown document with a version bump.
3. Mark a document `FROZEN` only after explicit stakeholder approval.
4. Preserve the previous version in Git history; never silently rewrite a frozen baseline.

No documents in this repository are currently FROZEN.
