# RMC MSA Analysis Tool

**Status:** Field & Screen Specification v0.3 updated with role/approval decisions — DRAFT / NOT FROZEN

A lightweight, web-based Measurement System Analysis (MSA) application for an initial single-company client of PT Ratama Mitra Kualitas. This repository starts with requirements and documentation; implementation and technology choices are not finalized.

## Purpose

Help authorized employees create Variable or Attribute MSA studies, enter measurements or OK/NG judgments, review calculations, track study history and revisions, and generate printable PDF reports—without the complexity of a general-purpose statistical package.

## Current scope

- **MVP:** Individual login, Admin/Supervisor/Operator roles, MSA Variable and Attribute, data entry, study lifecycle, history, audit trail, and PDF reporting.
- **Future:** Statistical Process Control (SPC), multi-client management, and advanced analytical methods.
- **Calculation direction:** Variable Average & Range with the Ratama 10×3×3 preset; Attribute Kappa vs Reference under review using the Ratama 50×3×2 workbook.
- **Calculation review:** [Final Calculation Review v0.2](docs/03-calculations/final-calculation-review-v0.2.md) documents cross-method differences, corrected test vectors and approval gates.
- **Pending validation:** Customer acceptance criteria, pooled Attribute Kappa methodology, review and freeze gates, study assumptions, alternative study sizes.

## Documentation

- [Field & Screen Specification v0.3](docs/04-ui-ux/field-and-screen-specification-v0.3.md): confirmed Supervisor approval/finalization and Attribute reference management; Operator all-history view; final PDF logo and authorized signature block; 35 UX checks. [v0.2 archive](docs/04-ui-ux/field-and-screen-specification-v0.2.md).

Start at [Documentation Index](docs/README.md), then read the current [Requirements Analysis v0.3](docs/01-discovery/requirements-analysis-v0.3.md), [Calculation Source Audit](docs/03-calculations/ratama-source-audit-v0.1.md), the [Variable Calculation Draft](docs/03-calculations/variable-msa-calculation-spec-v0.1.md), the [Attribute Audit](docs/03-calculations/attribute-msa-audit-v0.1.md), the [Calculation QA Plan](docs/06-quality-assurance/calculation-test-plan-v0.1.md), and the [Decision Log](docs/decisions/decision-log.md).

## Documentation status policy

`DRAFT` → `REVIEW` → `FROZEN` only with explicit approval. An item marked *agreed in review* is **not** automatically frozen. Record changed assumptions and decisions in the decision log. Do not commit client-sensitive records to this public repository.

## Development

No framework, database, hosting provider, or implementation details have been approved yet.
