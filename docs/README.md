# Documentation Index

**Project:** RMC MSA Analysis Tool  
**Current phase:** Requirements gathering and source calculation audit  
**Document baseline:** v0.3 — DRAFT / NOT FROZEN

## Directory map

| Folder | Purpose | Current status |
| --- | --- | --- |
| [01-discovery](01-discovery/requirements-analysis-v0.3.md) | Business context, proposed product scope, user flow, current analysis | DRAFT |
| [02-requirements](02-requirements/README.md) | Detailed functional and non-functional requirements | Pending |
| [03-calculations](03-calculations/README.md) | Audited source results, Variable draft formulas and Attribute candidate methods | DRAFT / review |
| [04-ui-ux](04-ui-ux/README.md) | User journeys, fields, screens and prototypes | Pending |
| [05-architecture](05-architecture/README.md) | Data model, API, authentication, infrastructure | Pending |
| [06-quality-assurance](06-quality-assurance/README.md) | Statistical regression plan, UAT and release criteria | DRAFT / pending implementation |
| [decisions](decisions/decision-log.md) | Decision register, approvals, outstanding questions | Active |

## Reading order

1. [Requirements Analysis v0.3 — DRAFT](01-discovery/requirements-analysis-v0.3.md)
2. [Decision Log](decisions/decision-log.md)
3. [Ratama source audit v0.1](03-calculations/ratama-source-audit-v0.1.md)\n4. [Variable calculation spec v0.1](03-calculations/variable-msa-calculation-spec-v0.1.md)\n5. [Attribute audit v0.1](03-calculations/attribute-msa-audit-v0.1.md)\n6. [Calculation QA plan v0.1](06-quality-assurance/calculation-test-plan-v0.1.md)\n7. Freeze only after the remaining approval gates pass.

## Approval rules

- `DRAFT`: proposals, assumptions, and feedback; not approved for implementation.
- `REVIEW`: candidate specification under stakeholder review.
- `FROZEN`: explicitly approved, versioned baseline.
- Agreement on an individual requirement does **not** freeze an entire document.
- Revise with a new version and log changes before replacing an approved baseline.

## Source materials

Legacy Ratama Excel workbooks and prior v1.0/v1.1 drafts are **reference inputs**, not governing specifications for this rebuild. Do not assume their formulas are automatically valid. Never upload confidential client data to this public repository.
