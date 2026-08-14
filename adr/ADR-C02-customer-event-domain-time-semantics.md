# ADR-C02: CustomerEvent Canonical Domain and Time Semantics

| Metadata | Value |
| --- | --- |
| Status | Accepted |
| Decision Owner | Carelog Product Main |
| Date | 2026-08-13 |
| Related Jira | RPL-88 |
| Related Repositories | `carelog-architecture`, `carelog-fe`, `carelog-be`, `harness-foundation-docs` |
| Depends On | Carelog frontend product model at the bootstrap revision; Foundation documentation ownership rules |
| Supersedes | None; this establishes the Carelog CustomerEvent product semantic without replacing legacy backend assets |
| Canonical Merge SHA | 4c3e48162557577ccc86c21f8c79de421ecbaf25 |

## Context

Carelog needs one product concept for scheduled and completed customer-facing activity. The frontend already expresses a CustomerEvent model and uses it for history and planned follow-up experiences. The backend source revision does not yet provide a production CustomerEvent API or persistence implementation.

The architecture must therefore accept the product semantic while explicitly separating it from backend completion and from existing backend journal/relation assets.

## Previous Canonical Reality

At the bootstrap source revision:

- `carelog-fe` defines `CustomerEvent` with `PLANNED`, `OCCURRED`, and `CANCELLED` statuses;
- planned events carry `scheduledAt`, occurred events carry `occurredAt`, and cancellation retains the original planned time context;
- overdue behavior is derived by comparing a planned event's scheduled time with the current time;
- the frontend uses CustomerEvent history for the timeline and planned events for the follow-up expression;
- `carelog-be` has legacy `Relation` and `RelationJournal` assets, including relation-scoped journal data and a separate journal status model;
- a production backend CustomerEvent API and persistence boundary are not implemented at this snapshot.

The existence of `RelationJournal` is not normalized into a claim that it is the same domain object as CustomerEvent.

## Decision

The canonical Carelog Product Domain is:

```text
User
  → Workspace
    → Customer
      → CustomerEvent
```

`CustomerEvent` is the product/API concept for a customer event. Its accepted status/time invariant is:

| Status | Canonical time meaning |
| --- | --- |
| `PLANNED` | The event is intended to happen at `scheduledAt`. |
| `OCCURRED` | The business occurrence is recorded at `occurredAt`. Planned context may remain available when the event came from a planned event. |
| `CANCELLED` | The planned event is cancelled while its original planned/event time context is retained. |

The accepted derived and projection semantics are:

- `overdue` is true only when `status = PLANNED` and `scheduledAt < now`.
- `OVERDUE` is not a CustomerEvent status enum value.
- `Timeline` is a history projection of CustomerEvent records.
- `Follow-up` is the P0 planned-CustomerEvent UX expression.
- `Handoff` is not an Accepted invariant in this ADR; its phase is Deferred.
- `Relation` and `RelationJournal` remain backend legacy assets. `RelationJournal == CustomerEvent` is not a 1:1 declaration.

The semantic is accepted independently of backend implementation:

```text
Accepted Product Domain  !=  Completed Backend Implementation
```

## Boundaries

Carelog Product Architecture owns CustomerEvent meaning, status/time invariants, and product projections. `carelog-fe` currently carries the canonical executable product model at the source snapshot. `carelog-be` will own the eventual executable API and persistence implementation when that work is separately defined.

This ADR does not define:

- production DTOs or API payloads;
- database tables, columns, or mappings;
- idempotency or event-delivery guarantees;
- timezone or wire timestamp representation;
- optimistic locking or concurrency behavior;
- a relation-to-event mapping;
- Shared Platform or Shared AI ownership.

## Consequences

- Product work can use one CustomerEvent concept across timeline and planned follow-up experiences.
- Frontend canonical behavior can be preserved while the backend gap remains visible instead of being hidden by a premature persistence mapping.
- Legacy relation/journal behavior can be migrated or adapted later without falsely asserting object identity.
- Consumers must treat overdue as a computed view, not as a stored lifecycle status.
- An implementation team still needs a separate backend contract and verification slice before claiming production support.

## Deferred Decisions

- Handoff phase and Handoff domain semantics;
- CustomerEvent production API and backend persistence implementation;
- exact DTO, database, idempotency, timezone, concurrency, and error contracts;
- event publication, replay, and cross-service delivery behavior;
- migration or adaptation strategy for `Relation` and `RelationJournal`;
- any relation between CustomerEvent and a future integration event envelope.

## Verification

- Read-only source verification confirmed the frontend status union and time-bearing types in `src/domain/customerEvent.ts`.
- Read-only source verification confirmed planned-to-occurred and planned-to-cancelled product transitions and derived overdue behavior in the frontend domain/store sources.
- Read-only source verification confirmed the Follow-up projection in `docs/design/carelog/README.md:35-36` and the legacy-to-CustomerEvent planned-event mapping in `src/adapters/legacyCustomerEventAdapter.ts:34-51`.
- Read-only source verification confirmed `RelationJournal` is a separate relation-backed backend entity.
- Read-only source verification found no production backend CustomerEvent API or persistence implementation at the bootstrap revisions.
- The source revisions used for this characterization are recorded in [README](../README.md).

## References

- `carelog-fe/src/domain/customerEvent.ts` at `carelog-fe/main@207fd3f5bb27241b7e858ce7cf888db6675eaaa8`.
- `carelog-fe/src/state/eventStore.ts` at `carelog-fe/main@207fd3f5bb27241b7e858ce7cf888db6675eaaa8`.
- `carelog-fe/docs/design/carelog/README.md:35-36` and `carelog-fe/src/adapters/legacyCustomerEventAdapter.ts:34-51` at `carelog-fe/main@207fd3f5bb27241b7e858ce7cf888db6675eaaa8`.
- `carelog-be/src/main/java/carelog/carelog/relation/domain/Relation.java` at `carelog-be/dev@2bae6677498aefe57b583f648519bd9f9c5ebc0a`.
- `carelog-be/src/main/java/carelog/carelog/journal/domain/RelationJournal.java` at `carelog-be/dev@2bae6677498aefe57b583f648519bd9f9c5ebc0a`.
- [Customer event lifecycle contract](../contracts/customer-event-lifecycle.md).
