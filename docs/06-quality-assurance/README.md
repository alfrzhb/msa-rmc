# Quality Assurance / UAT

**Status:** REVIEW DRAFT — implementation pending; no application regression suite has yet been run.

## Review-ready artifacts

- [Calculation QA & Regression Plan v0.1](calculation-test-plan-v0.1.md) — numerical golden values, data-validation cases, role/lifecycle safeguards.
- [Calculation Review Regression Matrix v0.2](calculation-review-test-matrix-v0.2.md) — Attribute pooled-vs-trial-average Kappa, sample-vs-decision denominator, NDC display edge and acceptance boundary tests.
- [Final Calculation Review v0.2](../03-calculations/final-calculation-review-v0.2.md) — method rationale, limitations and outstanding gates.
- [Source audit v0.1](../03-calculations/ratama-source-audit-v0.1.md) — original source reconciliation.

Do not publish raw client workbooks, appraiser identities or confidential study measurements to this public repository.

## Screen & field UX review

- [Field & Screen Specification v0.2](../04-ui-ux/field-and-screen-specification-v0.2.md) — 25 nonexecuted UI acceptance checks (UI-01 through UI-25) for progressive disclosure, edit permissions, separate blind reference, Supervisor review/finalization, results-first display, revision history and report snapshots.

## Field & Screen v0.3 product-owner decision tests

- [Field & Screen Specification v0.3](../04-ui-ux/field-and-screen-specification-v0.3.md) extends the UX review matrix to UI-35: Operator all-history read-only rules, Supervisor-only Attribute Known Reference management, explicit Approve & Finalize with actor/time, signed-logo PDF without invented assets, and refusal to certify a study under unapproved Quality rules. Tests are **specified, not yet implemented or run**.

## DB design review (not executed)

- [Database ERD v0.1](../05-architecture/erd-database-v0.1.md) records **DB-01 through DB-18** candidate tests: partial Draft saves, 90/300+50 completeness, composite revision FK, owner/RBAC isolation, stale results, Supervisor-only approval, R2/D1 failure paths and immutable historical PDF. These are design checks, not live D1 regression results.

## ERD v0.2 integrity test plan — not yet executed

- [Corrected Database ERD v0.2](../05-architecture/erd-database-v0.2.md) expands the schema review into **ER2-01 to ER2-20**: composite FK, Attribute blind disclosure, input digest, signer matching, approved policy, audit before/after, concurrent Finalize, D1 rollback, R2 orphan handling and report immutability. These checks require actual Drizzle/D1 integration tests.
