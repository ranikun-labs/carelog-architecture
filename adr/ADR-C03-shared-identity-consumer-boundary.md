# ADR-C03: Shared Identity Consumer Boundary

| Metadata | Value |
| --- | --- |
| Status | Proposed |
| Decision Owner | Carelog Product Main |
| Date | 2026-08-13 |
| Related Jira | RPL-88 |
| Related Repositories | `carelog-architecture`, `carelog-be`, `platform-services`, `harness-foundation-docs` |
| Depends On | Platform repository/service boundaries and identity ownership evidence; current Carelog Auth/Gateway transition source |
| Supersedes | None; this is a Carelog consumer-boundary decision and does not accept a shared wire schema |
| Canonical Merge SHA | 4c3e48162557577ccc86c21f8c79de421ecbaf25 |

## Context

Carelog needs a stable boundary for consuming authenticated identity without taking ownership of Shared Identity internals. The current Carelog backend contains an Auth/Gateway implementation path. `platform-services` contains the target Shared Gateway / Shared Identity runtime foundation. Those facts describe a transition; they do not by themselves establish the final platform contract.

## Previous Canonical Reality

At the bootstrap source revisions:

- `carelog-be` is the current Carelog backend implementation and the transition/extraction source for Auth/Gateway behavior;
- its current backend path accepts a trusted internal context and turns it into an authenticated principal used by Carelog authorization;
- `platform-services` contains independent Gateway and Identity runtime foundation applications, with extraction and cutover still separate work;
- Foundation documentation gives Shared Platform ownership of shared identity capabilities while keeping product authorization in the product service;
- no accepted trusted-context wire schema is available for Carelog to promote as the future canonical interface.

The current implementation source and the final platform canonical owner are distinct:

```text
Current Implementation Source  !=  Final Platform Canonical Owner
```

## Decision

The proposed Carelog boundary is semantic-only:

```text
Shared Platform Provider
        → authenticated identity/context
        → Carelog Consumer
        → Carelog Product authorization
```

Carelog Architecture owns:

- the semantic meaning of authenticated identity/context as consumed by Carelog;
- `identityUserId` and its linkage to Carelog domain data;
- Carelog-local Workspace/Product Membership meaning;
- Carelog Product authorization;
- Customer ownership;
- a candidate consumer behavior when Shared Identity is unavailable or an identity is deactivated;
- Carelog product-state ownership during transition and cutover.

Shared Platform owns:

- Account;
- Credential;
- provider-neutral OAuth;
- Token;
- Session;
- issuer and audience semantics;
- JWKS and signing-key publication semantics;
- Identity database schema;
- Redis/session semantics;
- the final trusted-context wire contract.

The product consumer must not turn a current trusted-header implementation into the final platform contract. The proposed semantic boundary candidate is a relationship, not an accepted header name, claim layout, transport, or session schema.

## Boundaries

Carelog consumes an authenticated identity and resolves it to Carelog-local linkage and authorization. It does not own the platform Account lifecycle, credentials, provider-neutral provider integration, token issuance, session lifecycle, key publication, or identity storage.

Shared Platform provides shared identity/runtime capabilities. It does not acquire Carelog Customer ownership or Carelog-local membership/authorization merely because it authenticates a principal.

During transition, `carelog-be` may remain the behavior/extraction source while `platform-services` becomes the target provider runtime. Source location is not a canonical ownership transfer.

The candidate behavior for unavailable or deactivated Shared Identity is intentionally not an Accepted protocol. A later decision must address fail-closed authorization for new or mutating product operations, stale context, and the distinction between read and mutation behavior.

## Consequences

- Carelog can change the provider implementation without moving Customer ownership or product authorization into Shared Platform.
- Shared Platform can serve multiple product consumers without adopting Carelog-specific domain identifiers or membership rules.
- The current backend trusted-context path can be preserved as transition evidence while a separately reviewed final wire contract is designed.
- Identity extraction and product-state cutover remain independently reviewable; a runtime foundation or source extraction does not prove consumer cutover.
- A proposed semantic boundary candidate is recorded here, but no Accepted Shared Identity contract is established; implementation teams cannot infer exact token, session, header, or deployment values from this ADR.

## Deferred Decisions

- final trusted-context wire schema and verification mechanism;
- exact header names or equivalent transport fields;
- exact JWT claim schema;
- issuer, audience, and JWKS values and rotation behavior;
- Identity versioning and deactivation protocol;
- web/mobile session transport and refresh policy;
- Identity database schema;
- Redis/session semantics and storage keys;
- final Shared Gateway route and hostname;
- production public host and cutover timing;
- exact NATS subjects;
- Shared AI provider/model;
- Shared AI DTO;
- exact Carelog behavior when Shared Identity is unavailable or deactivated.

## Verification

- Read-only source verification confirmed the current Carelog trusted-context implementation in `GatewayHeaderAuthFilter` and its principal mapping behavior.
- Read-only source verification confirmed independent Gateway and Identity runtime foundation applications in `platform-services`.
- Read-only Foundation verification confirmed that product authorization remains product-owned and that a trusted-context wire contract requires separate acceptance.
- This ADR remains Proposed. Current implementation evidence does not promote it to Accepted.
- Source revisions are recorded in [README](../README.md).

## References

- `carelog-be/src/main/java/carelog/carelog/auth/web/GatewayHeaderAuthFilter.java` at `carelog-be/dev@2bae6677498aefe57b583f648519bd9f9c5ebc0a`.
- `carelog-be/src/main/java/carelog/carelog/auth/app/GatewayUserDetails.java` at `carelog-be/dev@2bae6677498aefe57b583f648519bd9f9c5ebc0a`.
- `platform-services/gateway-app` and `platform-services/platform-core/identity` runtime foundation sources at `platform-services/main@853adce7515f921931d4731bef5fc43f75e56e3d`.
- `harness-foundation-docs/docs/architecture/repository-service-boundaries.md` at `harness-foundation-docs/main@69fceb267094756913f400f8b4045590bd0e8ac5`.
- `harness-foundation-docs/docs/contracts/backend-service-foundation/identity-token-contract.md` at the same revision; its Draft status is not promoted here.
- [Authenticated Carelog context contract](../contracts/authenticated-carelog-context.md).
