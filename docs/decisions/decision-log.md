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
| DEC-009 | Store evolving specifications as Markdown in GitHub | Reduce risk of losing requirements history | Documentation workflow |

## Open decisions (DISCUSSION REQUIRED)

| ID | Question | Current proposal |
| --- | --- | --- |
| OPEN-001 | Number of analysis methods per MSA type | Discuss Average & Range for Variable; Kappa vs Reference plus relevant indicators for Attribute |
| OPEN-002 | Flexible vs locked part/appraiser/trial counts | Old Ratama templates are candidates, not defaults officially approved |
| OPEN-003 | Authoritative formulas, calculation versions, rounding, edge cases | Independently audit and approve test vectors |
| OPEN-004 | Judgment thresholds and standards | Ratama/client confirmation required |
| OPEN-005 | Can Operators view all studies or only their own? | Pending |
| OPEN-006 | Exact Admin and Supervisor permissions, including deletion and revision | Proposed matrix needs detailed approval |
| OPEN-007 | Finalize and revision lifecycle details | Proposed, not FROZEN |
| OPEN-008 | Dashboard fields, detailed PDF format, branding | Proposed, not FROZEN |
| OPEN-009 | Technology stack, hosting, backup and security implementation | Decide after requirements and calculations |
| OPEN-010 | Attribute agreement metrics and reference-data requirements | Pending method review |

## Change control

1. Add new decisions or change proposals to this log.
2. Update the affected Markdown document with a version bump.
3. Mark a document `FROZEN` only after explicit stakeholder approval.
4. Preserve the previous version in Git history; never silently rewrite a frozen baseline.

No documents in this repository are currently FROZEN.
