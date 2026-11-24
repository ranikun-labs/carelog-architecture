# Visit History Evidence View Contract

| Contract posture | Value |
| --- | --- |
| Product semantic | Proposed — design default set by AI Main; final adoption by Carelog Product Main |
| Owner | Carelog Product Main (product), AI Main (data/safety behavior) |
| Scope | Customer detail "치료 경과": which visit facts are shown, how each links to its source record, and when the server re-checks access |
| Backend status | Not implemented. Production integration stays under RPL-62 `HOLD — BLOCKED_BACKEND_CONTRACT` |

## Purpose

A therapist opening one customer understands, per visit, the recorded symptom, what was actually performed, and the recorded response, and sees what is missing as a question for the next visit. Every fact is traceable to its visit and source record. The view never shows a result the current therapist is no longer allowed to see.

## Design default

- Initial load shows the **3 most recent completed visits**, newest first by actual visit time (`occurredAt`, see [CustomerEvent lifecycle](customer-event-lifecycle.md)). `PLANNED`/`CANCELLED` events are excluded.
- "이전 방문 더보기" appends the next page **below on the same screen**. The earlier 90-day / 5-visit proposal is kept as history only.

## Visit item fields

| Field | Meaning | Rule |
| --- | --- | --- |
| Recorded symptom | Customer statement recorded at that visit | Labelled as customer report (`CLIENT_REPORT`) |
| Performed action | What the therapist recorded as actually done | Only performed actions; a planned action is never shown as performed |
| Recorded response | Response recorded at that visit | Labelled `CLIENT_REPORT` or `THERAPIST_OBSERVATION`; never carried over from another visit |
| Unconfirmed | What the record does not state | Shown explicitly as 미확인; never inferred |
| Next question | Question derived from an unconfirmed item | Must cite the visit whose gap it comes from |
| Source | Visit date + source record reference | Reference is an opaque ID, never the record text or an access secret |

Numeric self-reports from different visits are shown per visit only; the view does not compare them or claim treatment effect, because time and conditions of measurement are unconfirmed.

## Server re-check points

Each of the following is a separate server request that re-checks the current therapist's Workspace, customer and visit access at the time of the request:

1. Initial visit list.
2. Each "더보기" page. Older visits are **not** preloaded and hidden on the client.
3. Each source record open ("원기록 보기"). The record text is fetched on open, **not** embedded in the list response. A link that was visible before access was revoked is refused on click.

On refusal the response reveals neither the record text nor whether the customer or visit exists. Permission granted at list time or holding a link is never sufficient.

## Late-response boundaries

Customer switch, Workspace switch, logout, source record edit and access revocation invalidate in-flight results. The client compares Workspace ID, customer ID and request generation before showing any response, and discards a mismatched one without overwriting the current view. If a revocation signal may have been missed, the client shows nothing until it re-validates. The four acceptance cases are in the synthetic evidence document (RPL-87 design packet).

## Rendering

All record text is rendered as text, never as HTML.

## Missing and Deferred

- Exact API/DTO shape, pagination cursor, error wire schema.
- Source revision handling and how revocation reaches an open client (needs Product, clinical and security owners).
- Which free JSONB fields of a treatment entry map to symptom / performed / response.
- Approved clinical evidence lookup for a selected question (follow-up phase).
- Excluded: external transfer of real patient records, AI auto-save, automated treatment decisions, the 10m walk experiment.

## Evidence

- Synthetic design case and acceptance cases: neck discomfort, `synthetic-visit-1` (2026-11-04) and `synthetic-visit-2` (2026-11-11). No real clinical record was read.
- Synthetic preview aligned to that case; not a production implementation.
