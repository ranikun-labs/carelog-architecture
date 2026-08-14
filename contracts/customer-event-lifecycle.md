# CustomerEvent Lifecycle Contract

| Contract posture | Value |
| --- | --- |
| Product semantic | Accepted through [ADR-C02](../adr/ADR-C02-customer-event-domain-time-semantics.md) |
| Owner | Carelog Product Main |
| Scope | Carelog CustomerEvent meaning, lifecycle semantics, and derived product views |
| Backend status | CustomerEvent production API/persistence is not implemented at the bootstrap snapshot |

## Canonical status and time invariant

| Status | Required product time meaning |
| --- | --- |
| `PLANNED` | `scheduledAt` is the planned event time. |
| `OCCURRED` | `occurredAt` is the time the business event occurred; planned context may be retained when present. |
| `CANCELLED` | The planned event is cancelled and its original planned/event time context is retained. |

There is no `OVERDUE` status. Overdue is derived only as:

```text
status = PLANNED && scheduledAt < now
```

The comparison is a product derivation, not a stored lifecycle transition.

## Create and transition semantics

- Creating a planned event establishes `PLANNED` with `scheduledAt`.
- Creating an already occurred event establishes `OCCURRED` with `occurredAt`.
- A planned event may be recorded as occurred; the resulting event has `OCCURRED` and `occurredAt`, while its planned context may be retained.
- A planned event may be cancelled; the resulting event has `CANCELLED` and retains the original planned time context.
- The contract does not infer additional transitions, edit rules, or retry behavior beyond these product semantics.

## Derived product views

- `Timeline` is a history projection of CustomerEvent records.
- `Follow-up` is the P0 UX expression of planned CustomerEvent records.
- `Handoff` is not an Accepted CustomerEvent invariant in this slice. Its phase and relationship to CustomerEvent are Deferred.

## Source-of-truth ownership

CustomerEvent is the Carelog Product/API concept. The architecture repository owns the semantic contract. The frontend source snapshot is the current canonical executable product model. The backend repository owns the future executable API and persistence implementation when that work is separately accepted.

The backend `Relation` and `RelationJournal` assets remain legacy implementation assets. This contract does not map either asset 1:1 to CustomerEvent.

## Missing and Deferred

The following are intentionally absent:

- production API and DTO schema;
- database mapping or migration;
- idempotency contract;
- timezone and timestamp wire representation;
- optimistic locking or concurrency contract;
- exact error behavior;
- event publication, replay, or integration delivery contract;
- Handoff phase and domain semantics;
- Relation/RelationJournal migration strategy.

## Evidence

- `carelog-fe/src/domain/customerEvent.ts` at `carelog-fe/main@207fd3f5bb27241b7e858ce7cf888db6675eaaa8` defines the status union, time fields, derived overdue function, and planned transitions.
- `carelog-fe/src/state/eventStore.ts` at the same revision exposes product creation and planned-event transition behavior.
- `carelog-be/src/main/java/carelog/carelog/journal/domain/RelationJournal.java` at `carelog-be/dev@2bae6677498aefe57b583f648519bd9f9c5ebc0a` remains a separate relation-backed journal entity.
- [ADR-C02](../adr/ADR-C02-customer-event-domain-time-semantics.md) records the Accepted semantic and the backend implementation gap.
