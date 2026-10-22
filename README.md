# RMC MSA Analysis Tool

**Status:** Discovery / Requirements Analysis (DRAFT — NOT FROZEN)

A lightweight, web-based Measurement System Analysis (MSA) application for an initial single-company client of PT Ratama Mitra Kualitas. This repository starts with requirements and documentation; implementation and technology choices are not finalized.

## Purpose

Help authorized employees create Variable or Attribute MSA studies, enter measurements or OK/NG judgments, review calculations, track study history and revisions, and generate printable PDF reports—without the complexity of a general-purpose statistical package.

## Current scope

- **MVP:** Individual login, Admin/Supervisor/Operator roles, MSA Variable and Attribute, data entry, study lifecycle, history, audit trail, and PDF reporting.
- **Future:** Statistical Process Control (SPC), multi-client management, and advanced analytical methods.
- **Pending validation:** Calculation methods, formulas, acceptance criteria, number of samples/trials, and several permission details.

## Documentation

Start at [Documentation Index](docs/README.md), then read the current [Requirements Analysis v0.3](docs/01-discovery/requirements-analysis-v0.3.md) and [Decision Log](docs/decisions/decision-log.md).

## Documentation status policy

`DRAFT` → `REVIEW` → `FROZEN` only with explicit approval. An item marked *agreed in review* is **not** automatically frozen. Record changed assumptions and decisions in the decision log. Do not commit client-sensitive records to this public repository.

## Development

No framework, database, hosting provider, or implementation details have been approved yet.
