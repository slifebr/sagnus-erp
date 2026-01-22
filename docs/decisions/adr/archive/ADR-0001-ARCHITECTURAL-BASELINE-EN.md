# ADR-0001 – Sagnus ERP Architectural Baseline

## Status
ACCEPTED

## Date
2026-01-19

## Context
Sagnus ERP is the modernization of a legacy ERP based on Oracle Forms 6i,
with strong Oracle database dependency and business logic spread across Forms and PL/SQL.

The goal is to evolve into a modern, sustainable, long-term Web platform
with technological independence and architectural clarity.

## Decision
The official Sagnus ERP architecture is defined by the following principles:

- DDD (Domain-Driven Design) + Hexagonal Architecture
- Maven multi-module monorepo
- One Bounded Context per module
- Domain implemented in pure Java
  - No Spring
  - No JPA
  - No frameworks
- Repositories defined as PORTS in the domain
- Application Layer orchestrates use cases
- Infrastructure Layer implements technical ADAPTERS
- Controllers contain no business logic
- Gateway acts as a thin BFF
- GraphQL-first strategy
- REST only when technically justified
- Contracts isolated in `sagnus-bc-contracts-*` modules
- Database agnostic (Oracle and PostgreSQL)

## Consequences

### Becomes mandatory
- Respect ports & adapters
- Keep the domain pure
- Avoid direct coupling between BCs
- Version contracts when required

### Requires a new ADR
- Creation or split of Bounded Contexts
- Introducing exceptions to pure domain
- Changing Gateway or API strategy
- Introducing new cross-cutting patterns

## Notes
This ADR defines the architectural baseline.
All future evolution must start from this baseline.

## References
- .ai-context/
- docs/CHECKLIST_MERGE.md
- docs/CONVENTIONAL_COMMITS.md
