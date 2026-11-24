# Carelog Architecture Foundation

This repository is the canonical home for Carelog-specific product architecture, ADRs, and cross-repository contracts. It records product meaning and ownership boundaries; it does not replace executable implementation repositories or the platform-wide constitution.

## Canonical authority contract

| Repository or system | Canonical responsibility |
| --- | --- |
| `harness-foundation-docs` | Ranikun Labs platform-wide Architecture Constitution and shared foundation rules |
| `carelog-architecture` | Carelog-specific Product Architecture, ADRs, and cross-repository contracts |
| `carelog-fe` | Carelog Frontend executable implementation |
| `carelog-be` | Carelog Backend executable implementation and transition-period behavior/extraction source |
| `platform-services` | Shared Gateway / Shared Identity executable implementation and target Shared Platform provider runtime |
| Jira | Work ownership and delivery status |
| Confluence | Projection and navigation |

`platform-services` is not a child implementation of this repository. The intended relationship is:

```text
Carelog Product Consumer  ↔  Shared Platform Provider
```

Product architecture owns the Carelog meaning of a consumed identity and product authorization. Shared Platform owns the platform identity and shared runtime semantics. Neither ownership is inferred from repository adjacency.

## Document map

| Area | Document |
| --- | --- |
| System context | [System context](architecture/system-context.md) |
| Ownership | [Ownership map](architecture/ownership-map.md) |
| Runtime state | [Current and target runtime topology](architecture/runtime-topology.md) |
| Customer ownership | [ADR-C01](adr/ADR-C01-customer-lifecycle-workspace-ownership.md), [customer ownership contract](contracts/customer-ownership.md) |
| Customer events | [ADR-C02](adr/ADR-C02-customer-event-domain-time-semantics.md), [customer event lifecycle contract](contracts/customer-event-lifecycle.md) |
| Identity boundary | [ADR-C03](adr/ADR-C03-shared-identity-consumer-boundary.md), [authenticated Carelog context contract](contracts/authenticated-carelog-context.md) |
| Visit history evidence view | [visit history evidence view contract](contracts/visit-history-evidence-view.md) (Proposed; RPL-62 HOLD) |

Decision status and implementation status remain separate. In particular, an Accepted product invariant does not prove that the backend API or persistence implementation is complete.

## Bootstrap source snapshot

The following revisions are implementation and architecture evidence for this bootstrap. A SHA is a reproducibility anchor, not a permanent business invariant.

| Source | Revision |
| --- | --- |
| `carelog-be/dev` | `2bae6677498aefe57b583f648519bd9f9c5ebc0a` |
| `carelog-fe/main` | `207fd3f5bb27241b7e858ce7cf888db6675eaaa8` |
| `platform-services/main` | `853adce7515f921931d4731bef5fc43f75e56e3d` |
| `harness-foundation-docs/main` | `69fceb267094756913f400f8b4045590bd0e8ac5` |
| `carelog-architecture` seed/main | `dee955ef23dcdd6b77c98c1e53c5e271339b3b1e` |
| Jira | `RPL-88` |

## Existing Carelog documentation disposition

Existing `carelog-be/docs/**` documents remain in place. This repository references their evidence where useful; this slice does not move, delete, or rewrite them.

| Disposition | Existing material |
| --- | --- |
| KEEP | `ARCHITECTURE.md`, `DomainArchitecture.md`, `ERD_v1.md`, `SoftwareEngineeringPrinciples.md`, `carelog-architecture-review.md`, `soft-delete-audit-review.md`, `context.md`, `context/scg-monorepo/carelog-be.md`, `oauth-state-product-client-binding-contract.md` |
| REFERENCE | `foundation/foundation-context.md`, `scg-auth-design.md`, `scg-implementation-plan.md`, SCG troubleshooting documents, `operations/mac-mini-pilot-readiness.md` |
| MIGRATE_LATER | Carelog-specific runtime profile portion of `ADR-001`, `runtime-deployment-profiles.md` |
| DEPRECATE_LATER | `MSA-Carelog.md`, `ADR-002` platform communication document, `finance-frontend-shared-identity-contract.md`, `monorepo-strategy.md` |

These labels describe this architecture bootstrap's reference posture. They do not authorize a migration or deprecation change. `MSA-Carelog.md` is `DEPRECATE_LATER` because its legacy Schedule model uses `SCHEDULED`, `OVERDUE`, `COMPLETED`, and `DEFAULTED`, which conflicts with the Accepted CustomerEvent model of `PLANNED`, `OCCURRED`, and `CANCELLED` with overdue derived only. The legacy document remains unmodified and is not a new Product Canonical Source.

## Deliberate scope boundary

This foundation does not define production DTOs, database mappings, token or session schemas, exact public routes or hosts, exact event-bus subjects, or deployment and production-activation decisions. Those values remain Deferred in the documents that depend on them.

The current backend Customer implementation and current trusted-context implementation are recorded as transition evidence. They are not silently normalized into a final Shared Platform contract.
