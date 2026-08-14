# Carelog Ownership Map

## Canonical ownership

| Concern | Canonical owner | Executable or projecting surface | Boundary note |
| --- | --- | --- | --- |
| Platform-wide architecture constitution | `harness-foundation-docs` | Foundation documents | Shared rules are referenced, not copied into a product implementation source |
| Carelog product architecture and ADRs | `carelog-architecture` | This repository | Owns Carelog-specific meaning and cross-repository contract boundaries |
| Customer lifecycle and workspace ownership | `carelog-architecture` | `carelog-be` transition/current implementation; `carelog-fe` consumer UI | Product-local; Workspace and current Organization terminology are not permanently equated here |
| CustomerEvent domain and time semantics | `carelog-architecture` | `carelog-fe` canonical product model; backend implementation gap remains | Accepted meaning is not the same as completed backend API/persistence |
| Carelog authenticated-context consumption | `carelog-architecture` | `carelog-be` transition source; future Carelog consumer | Owns semantic consumption and product authorization, not shared identity internals |
| Shared Gateway / Shared Identity runtime | `platform-services` | Shared executable runtime | Provider to Carelog; not a child implementation of this repository |
| Account, Credential, provider-neutral OAuth, Token, Session, and identity persistence semantics | Shared Platform boundary | `platform-services` target provider runtime | Carelog consumes a stable semantic boundary; final wire/schema details remain Deferred |
| Frontend behavior and product UX | `carelog-fe` | Frontend source | Implements the product model and projections; does not define platform ownership |
| Backend behavior and transition/extraction source | `carelog-be` | Backend source | Current behavior is evidence; extraction source is not automatically final canonical owner |
| Work ownership and delivery status | Jira | Jira issue `RPL-88` | Status projection does not change code or architecture ownership |
| Navigation and projection | Confluence | Confluence pages | Projection does not become the canonical decision source |

## Ownership rules

1. A product decision is recorded here when it defines Carelog meaning or a Carelog cross-repository boundary.
2. A platform-wide decision is recorded in `harness-foundation-docs`; this repository may apply it to Carelog but does not absorb platform ownership.
3. `carelog-be` is the current implementation and transition/extraction source for backend behavior. That source status does not make Carelog backend code the final Shared Platform owner.
4. `platform-services` is the Shared Platform provider runtime. It must not be described as a subordinate implementation of `carelog-architecture`.
5. Carelog-local membership/authorization is not a platform-wide membership, subscription, or entitlement contract.
6. Implementation completion, runtime support, fixture status, product release, and ADR status must be reported separately.

## Ownership exclusions for this slice

This slice does not assign Carelog ownership of platform Account or Credential data, provider-neutral OAuth, Token or Session semantics, signing-key publication, identity persistence, shared runtime deployment, exact message subjects, or production activation.

It also does not establish a permanent `Workspace` to `Organization` equivalence. `Organization` is the current backend term; `Workspace` is the proposed Carelog product term whose permanent relationship remains Deferred.

## Evidence posture

The map is grounded in the bootstrap snapshot recorded in [README](../README.md). Source revisions are implementation evidence and may move independently of these product invariants. A later source change requires a new review of the affected contract; it does not retroactively rewrite the previous reality recorded by the ADRs.
