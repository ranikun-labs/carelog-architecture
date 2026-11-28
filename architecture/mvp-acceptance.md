# Carelog DEV MVP acceptance

Status: acceptance target; end-to-end execution is **unverified**. The Seoul DEV handoff in `harness-foundation-docs/docs/handoffs/2026-11-28-seoul-dev-mvp-handoff.md` records deployment evidence and open checks. This checklist does not approve deployment.

- A new user can sign up without an invitation and log in with a password.
- The signed-in user can create, read, update and delete a customer and a schedule. Saved data remains available after refresh and login again.
- A second account cannot read or modify the first account's customer or schedule through either the UI or direct requests; the server enforces the boundary.
- Check failures and access denial as well as the successful path, and record environment, date and evidence before marking any item verified.

The intended public frontend target is the Carelog DEV Cloudflare Worker. Its actual Worker name, domain, Gateway connection and signup/login request limits are **unverified** and must be confirmed through the Cloudflare owner or an approved read path before deployment. Do not infer a working route from this target.

AI implementation and verification remain with AI Main. Use the [central AI MVP handoff](https://github.com/ranikun-labs/harness-foundation-docs/tree/e71cb3d14eee230d4efcccaccf8a7adeeb7b2523/docs/handoffs/2026-11-27-ai-mvp) and the Seoul DEV handoff for later reports and current restrictions; this checklist does not duplicate or certify AI acceptance.
