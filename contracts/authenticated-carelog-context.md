# Authenticated Carelog Context Contract

| Contract posture | Value |
| --- | --- |
| Contract kind | Semantic-only consumer contract |
| Related decision | Proposed [ADR-C03](../adr/ADR-C03-shared-identity-consumer-boundary.md) |
| Provider | Shared Platform Provider |
| Consumer | Carelog Product Consumer |
| Wire/schema status | Deferred; no final transport or field schema is accepted |

## Participants and flow

```text
Shared Platform Provider
        |
        | authenticates and supplies verified identity/context
        v
Carelog Consumer
        |
        | resolves local linkage and checks product authority
        v
Carelog Product authorization
        |
        v
Workspace -> Customer -> CustomerEvent
```

## Provider semantic

The Shared Platform Provider is responsible for authenticating the platform identity and supplying a context that the final platform contract will define and protect. The provider owns platform Account, Credential, provider-neutral OAuth, Token, Session, identity persistence, issuer/audience semantics, key publication, and shared session semantics.

This contract does not define how that context is transported, which fields it contains, how it is versioned, or how a client session reaches the provider.

## Carelog consumer semantic

The Carelog Consumer must:

1. accept only an authenticated context that satisfies the final provider/consumer trust contract;
2. represent the stable platform identity as `identityUserId` at the Carelog semantic boundary;
3. resolve `identityUserId` to Carelog-local domain linkage without making it a Customer identifier;
4. evaluate Carelog-local Workspace/Product Membership and product authorization;
5. enforce Customer ownership and the cross-workspace prohibition;
6. keep product state and product authorization under Carelog ownership;
7. treat `identityUserId`, or an equivalent stable identifier, as a logical/weak reference: do not read or write the Shared Identity database or tables directly, and do not create a physical cross-service foreign key from Carelog storage to Shared Identity storage. Identity-related information is consumed only through an accepted API, event, token, or authenticated-context contract.

The Shared Platform identity is an input to Carelog authorization, not a replacement for Carelog-local authorization.

## Transition evidence

The current `carelog-be` Auth/Gateway path uses a trusted internal context and maps it to a Carelog authenticated principal. That implementation is evidence of the transition state only. It is not the final consumer contract, and its current header names, validation value, claim assumptions, and transport must not be promoted by this semantic document.

`platform-services` is the target Shared Platform provider runtime, while `carelog-be` remains the current implementation and extraction source during transition. Therefore:

```text
Current Implementation Source  !=  Final Platform Canonical Owner
```

## Unavailable or deactivated identity

The behavior when Shared Identity is unavailable or an identity is deactivated is a candidate follow-up decision, not an Accepted rule in this contract. The follow-up must explicitly decide:

- whether new and mutating product operations fail closed;
- whether any read operation may use previously verified context;
- how stale context is bounded;
- how product state is protected during provider failure or deactivation propagation.

Until that decision is accepted, no consumer may infer a bypass, grace period, or silent local activation from this contract.

## Missing and Deferred

- final trusted-context wire schema;
- exact header names or alternative transport fields;
- exact JWT claim schema;
- issuer, audience, and JWKS values;
- identity versioning and deactivation protocol;
- web/mobile session transport and refresh policy;
- Identity database schema;
- Redis/session semantics;
- final Gateway route and hostname;
- production public host and cutover timing.

The exact NATS subjects, Shared AI provider/model, and Shared AI DTO are also outside this identity contract and remain Deferred. No value for them is inferred here.

## Evidence

- `carelog-be/src/main/java/carelog/carelog/auth/web/GatewayHeaderAuthFilter.java` and `GatewayUserDetails.java` at `carelog-be/dev@2bae6677498aefe57b583f648519bd9f9c5ebc0a` provide transition evidence only.
- Independent Gateway and Identity runtime foundation sources are present in `platform-services/main@853adce7515f921931d4731bef5fc43f75e56e3d`.
- Foundation identity-token documentation is Draft and does not supply an Accepted Carelog wire schema at `harness-foundation-docs/main@69fceb267094756913f400f8b4045590bd0e8ac5`.
- Foundation repository/service-boundary guidance at `harness-foundation-docs/docs/architecture/repository-service-boundaries.md` assigns each service its own data and prohibits direct service-database access and cross-service foreign keys at `harness-foundation-docs/main@69fceb267094756913f400f8b4045590bd0e8ac5`.
- [ADR-C03](../adr/ADR-C03-shared-identity-consumer-boundary.md) owns the proposed boundary and its explicit deferrals.
