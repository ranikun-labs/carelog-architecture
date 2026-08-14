# ADR-C01: Customer Lifecycle and Workspace Ownership

| Metadata | Value |
| --- | --- |
| Status | Proposed |
| Decision Owner | Carelog Product Main |
| Date | 2026-08-13 |
| Related Jira | RPL-88 |
| Related Repositories | `carelog-architecture`, `carelog-be`, `carelog-fe`, `platform-services`, `harness-foundation-docs` |
| Depends On | Platform-wide ownership boundaries from `harness-foundation-docs`; current Carelog Customer behavior in `carelog-be`; current Carelog product scope in `carelog-fe` |
| Supersedes | None; this is a Carelog-specific decision and does not supersede Foundation records |
| Canonical Merge SHA | 4c3e48162557577ccc86c21f8c79de421ecbaf25 |

## Context

Carelog needs a product-owned rule for who may create, read, and mutate a Customer. The shared platform can authenticate a principal, but it must not decide Carelog Customer ownership or import a platform-wide commerce concept into the Carelog product boundary.

The current backend behavior is organization-scoped. The authenticated principal supplies the organization scope used for Customer creation and lookup, and Customer mutations require authentication plus Carelog-local authorization. The client does not choose a different organization for the create operation.

## Previous Canonical Reality

At the bootstrap source revision, the Carelog backend:

- treats Customer as part of the Carelog product scope;
- stores the current ownership term as `Organization` scope;
- derives create ownership from the authenticated principal scope rather than a client-supplied organization value;
- restricts lookup and mutation to the principal's organization scope and Carelog-local role authorization;
- rejects missing organization scope and does not resolve a Customer across organization boundaries.

This ADR records that observed reality without rewriting the older `Organization` terminology as if it had always meant `Workspace`.

## Decision

The following Carelog product rule is Proposed:

1. A Customer is owned within one Carelog Product Workspace boundary.
2. Customer creation ownership is derived from the authenticated principal's Carelog authorization scope. A client-supplied workspace or organization value is not an authority source.
3. Customer reads and mutations require authentication and Carelog-local product authorization.
4. A Customer lookup or mutation across workspace boundaries is prohibited, including when a caller knows the Customer identifier.
5. `Carelog Product Membership` means only Carelog-local workspace membership/authorization for this architecture. It does not mean Shared Commerce membership, platform-wide membership, subscription, or entitlement.
6. The current backend `Organization` term remains valid as implementation evidence. `Workspace` is the proposed Carelog product term; their permanent relationship is not decided here.

The decision is intentionally product-local. Shared Identity establishes an authenticated identity; Carelog determines whether that identity has authority in the relevant Carelog workspace.

## Boundaries

Carelog Product Architecture owns:

- Customer product meaning and lifecycle ownership;
- the semantic workspace scope used to enforce ownership;
- Carelog-local membership/authorization meaning;
- the prohibition on cross-workspace access;
- the product authorization decision for Customer mutation.

This ADR does not own or define:

- platform-wide membership;
- Shared Commerce, subscription, or entitlement semantics;
- Account, Credential, provider-neutral OAuth, Token, Session, or identity persistence semantics;
- a final Workspace-to-Organization vocabulary contract;
- an API, DTO, database, concurrency, or error wire schema.

## Consequences

- Customer authority remains explainable from the authenticated principal and Carelog-local authorization rather than from client input.
- Existing organization-scoped behavior can be preserved during transition without claiming that `Organization` is permanently identical to `Workspace`.
- Future identity extraction can change the authentication provider without moving Customer ownership into Shared Platform.
- Product Membership stays narrow, reducing the risk that commerce or platform entitlement decisions leak into Carelog architecture.
- Backend and frontend implementations must converge on the product semantic, but this ADR does not by itself implement or migrate either repository.

## Deferred Decisions

- permanent Workspace-to-Organization terminology;
- final Carelog Product Membership lifecycle;
- whether any platform-wide membership exists and how it relates to Carelog;
- subscription and entitlement semantics;
- exact Customer API/DTO, database, concurrency, and error contracts;
- any production migration or cutover procedure.

## Verification

- Read-only source verification confirmed the backend Customer service derives create, lookup, and update scope from the authenticated principal's current organization scope.
- Read-only source verification confirmed the backend Customer controller applies Carelog-local role authorization before delegating product operations.
- The source revisions used for this characterization are recorded in [README](../README.md).
- This ADR remains Proposed; source evidence does not promote it to Accepted.

## References

- `carelog-be/src/main/java/carelog/carelog/customer/app/CustomerServiceImpl.java` at `carelog-be/dev@2bae6677498aefe57b583f648519bd9f9c5ebc0a`.
- `carelog-be/src/main/java/carelog/carelog/customer/web/CustomerController.java` at `carelog-be/dev@2bae6677498aefe57b583f648519bd9f9c5ebc0a`.
- `carelog-be/docs/ERD_v1.md` and `carelog-be/docs/ARCHITECTURE.md` at `carelog-be/dev@2bae6677498aefe57b583f648519bd9f9c5ebc0a`.
- [Customer ownership contract](../contracts/customer-ownership.md).
- [Ownership map](../architecture/ownership-map.md).
