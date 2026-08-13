# Carelog System Context

## Purpose

This document fixes the authority and dependency context for Carelog architecture work. It distinguishes the repository that defines a rule from the repository that executes it.

## Authority and implementation context

```text
                         platform-wide constitution
                         + shared foundation rules
                    +-------------------------------+
                    |       harness-foundation-docs  |
                    +-------------------------------+
                                      |
                                      | constrains shared boundaries
                                      v
                    +-------------------------------+
                    |      carelog-architecture       |
                    | Carelog product architecture,   |
                    | ADRs, and cross-repo contracts  |
                    +-------------------------------+
                         |                     |
              executable product        consumes shared context
                         |                     |
             +-----------v----------+   +------v----------------+
             |       carelog-fe     |   |    platform-services    |
             | Frontend implementation|   | Shared Gateway /       |
             +-----------+----------+   | Shared Identity runtime |
                         |              +------------+------------+
                         |                           |
                         +-------------+-------------+
                                       v
                              +------------------+
                              |    carelog-be    |
                              | Backend product  |
                              | implementation   |
                              +------------------+

      Jira = work ownership / delivery status
      Confluence = projection / navigation
```

The diagram shows authority and repository relationships, not a claim that all runtime paths are already cut over. The current and target runtime states are separated in [runtime topology](runtime-topology.md).

## Canonical relationship

```text
Carelog Product Consumer
        |
        | consumes authenticated identity/context and applies product rules
        v
Shared Platform Provider
```

`platform-services` is not a lower-level implementation owned by `carelog-architecture`. Carelog architecture owns what Carelog means by an authenticated principal, local product membership, customer ownership, and product authorization. Shared Platform owns the platform identity and shared runtime capabilities that Carelog consumes.

## Context boundaries

| Boundary | Carelog architecture owns | Other canonical owner |
| --- | --- | --- |
| Product identity use | `identityUserId` meaning in Carelog and its linkage to Carelog domain data | Shared Platform owns the identity account and credential semantics |
| Workspace and customer | Carelog-local workspace membership/authorization and Customer ownership | No platform-wide membership or commerce decision is imported here |
| Customer events | CustomerEvent meaning, status/time invariants, and product projections | Carelog Backend will implement persistence and API later |
| Shared runtime | Consumer semantic and product authorization behavior | `platform-services` owns Shared Gateway / Shared Identity executable implementation |
| Platform constitution | Carelog-specific application of shared rules | `harness-foundation-docs` |
| Delivery and navigation | References and status projections | Jira and Confluence respectively |

## Status discipline

The following dimensions are intentionally independent:

- an ADR may be Proposed or Accepted;
- a contract may be semantic-only while its wire representation is Deferred;
- a frontend model may be canonical while backend production implementation is missing;
- an executable runtime may be a transition source or a target provider runtime;
- Jira status may describe delivery work without changing architecture ownership.

## References

- Carelog backend evidence: `carelog-be/docs/ARCHITECTURE.md`, `carelog-be/docs/DomainArchitecture.md`, `carelog-be/docs/carelog-architecture-review.md` at `carelog-be/dev@2bae6677498aefe57b583f648519bd9f9c5ebc0a`.
- Carelog frontend evidence: `carelog-fe/src/domain/customerEvent.ts` and `carelog-fe/src/state/eventStore.ts` at `carelog-fe/main@207fd3f5bb27241b7e858ce7cf888db6675eaaa8`.
- Shared platform evidence: `platform-services/README.md` and the independent Gateway/Identity runtime sources at `platform-services/main@853adce7515f921931d4731bef5fc43f75e56e3d`.
- Foundation authority evidence: `harness-foundation-docs/docs/architecture/repository-service-boundaries.md` and `docs/adr/ADR-0017-shared-platform-gateway-identity-physicalization.md` at `harness-foundation-docs/main@69fceb267094756913f400f8b4045590bd0e8ac5`.
