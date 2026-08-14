# Customer Ownership Contract

| Contract posture | Value |
| --- | --- |
| Product semantic | Proposed through [ADR-C01](../adr/ADR-C01-customer-lifecycle-workspace-ownership.md) |
| Owner | Carelog Product Main |
| Scope | Carelog-local Customer ownership and mutation authority |
| Wire/schema status | Not defined by this contract |

## Product semantic

A Carelog Customer belongs to one Carelog Product Workspace. Workspace scope is a product ownership boundary: it determines which Carelog-local principal may discover or mutate that Customer.

`Carelog Product Membership` means only Carelog-local workspace membership/authorization. It is not a platform-wide membership, Shared Commerce relationship, subscription, or entitlement.

## Ownership invariants

### Workspace-scoped ownership

Customer reads and mutations are evaluated inside the caller's Carelog workspace scope. A Customer identifier is not sufficient authority by itself.

### Principal-derived authority

The authenticated principal supplies the authority scope used for Customer ownership. Customer creation must not accept a client-supplied organization or workspace value as the source of authority.

### Cross-workspace prohibition

If a Customer is outside the authenticated principal's Carelog workspace scope, the Customer is not accessible for the requested product operation. Cross-workspace reads and mutations are prohibited.

### Mutation authority

Customer mutation requires both:

1. an authenticated principal/context;
2. Carelog-local product authorization for the relevant workspace and Customer.

The Shared Platform authenticates the identity; Carelog makes the product authorization decision.

## Terminology

The current backend implementation uses `Organization` for the scope that is attached to the authenticated principal and used by Customer operations. This contract uses `Workspace` as the proposed Carelog product term.

The contract does not declare that Workspace and Organization are permanently identical. That vocabulary decision remains Deferred.

## Missing and Deferred

This contract intentionally does not define:

- final Product Membership lifecycle;
- platform-wide membership;
- subscription or entitlement semantics;
- exact API or DTO shape;
- database or persistence mapping;
- concurrency behavior;
- error wire schema;
- a final identity or trusted-context transport.

## Evidence

- `carelog-be/src/main/java/carelog/carelog/customer/app/CustomerServiceImpl.java` at `carelog-be/dev@2bae6677498aefe57b583f648519bd9f9c5ebc0a` derives Customer scope from the authenticated organization value and queries by that scope.
- `carelog-be/src/main/java/carelog/carelog/customer/web/CustomerController.java` at the same revision requires authenticated Carelog role authorization for Customer operations.
- [ADR-C01](../adr/ADR-C01-customer-lifecycle-workspace-ownership.md) records the previous Organization terminology and proposed Workspace decision without retroactive normalization.
