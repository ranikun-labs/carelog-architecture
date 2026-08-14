# Carelog Runtime Topology

Current implementation topology and target architecture are intentionally separate. The target is a direction and ownership boundary, not proof that cutover or production activation has completed.

## CURRENT — implementation and transition state

```text
Carelog frontend executable
          |
          v
Carelog backend executable (`carelog-be`)
  - Carelog product implementation
  - Auth / Gateway transition implementation source
  - current product authorization and Customer behavior

Shared platform repository (`platform-services`)
  - independent Shared Gateway runtime foundation
  - independent Shared Identity runtime foundation
  - extraction / cutover work is in progress
```

Current evidence at the bootstrap revisions:

- `carelog-be` contains the Carelog backend and the Auth/Gateway transition source. Its current Customer behavior derives organization scope from the authenticated principal and applies Carelog-local authorization.
- `platform-services` contains independent Gateway and Identity runtime foundation applications. The foundation is not itself the completed Carelog traffic cutover or completed Shared Identity business implementation.
- The current trusted-context implementation in `carelog-be` is transition evidence. It is not the final platform wire contract.

The current state preserves the distinction:

```text
Current Implementation Source  !=  Final Platform Canonical Owner
```

## TARGET — product consumer and shared provider

```text
Carelog frontend
       |
       v
Carelog backend product consumer
       |
       | consumes authenticated identity/context
       v
Shared Platform provider
  - Shared Gateway
  - Shared Identity
       |
       v
Carelog-local product authorization
       |
       v
Workspace -> Customer -> CustomerEvent
```

In the target relationship, Carelog remains the owner of Customer ownership, Carelog-local membership/authorization, and CustomerEvent product semantics. Shared Platform remains the owner of Account, Credential, provider-neutral OAuth, Token, Session, identity persistence, and shared runtime semantics.

## Target boundary consequences

- Carelog consumes authenticated identity/context; it does not own the shared identity database or shared session semantics.
- Carelog resolves `identityUserId` to Carelog-local domain linkage and performs product authorization.
- Shared Platform does not acquire Carelog Customer, Timeline, Follow-up, Handoff, or CustomerEvent business ownership merely by providing authentication.
- An Accepted product domain contract does not imply that the target backend API or persistence exists.

## Deferred topology decisions

The following remain Deferred and are not inferred by the target diagram:

- final Shared Gateway route;
- Shared Identity hostname;
- exact public route, public host, or deployment topology;
- final trusted-context wire schema;
- web/mobile session transport and refresh policy;
- issuer, audience, and public-key publication details;
- identity database and Redis/session semantics;
- exact event-bus subjects;
- production public host and production cutover timing.

## References

- Current Carelog implementation: `carelog-be/dev@2bae6677498aefe57b583f648519bd9f9c5ebc0a`.
- Current frontend product model: `carelog-fe/main@207fd3f5bb27241b7e858ce7cf888db6675eaaa8`.
- Shared runtime foundation: `platform-services/main@853adce7515f921931d4731bef5fc43f75e56e3d`.
- Platform architecture authority: `harness-foundation-docs/main@69fceb267094756913f400f8b4045590bd0e8ac5`.
