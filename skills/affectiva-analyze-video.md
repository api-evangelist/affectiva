---
name: affectiva-analyze-video
description: Submit a video to the Affectiva Facial Coding API and retrieve frame-by-frame facial expression metrics.
api: Affectiva Facial Coding API
base_url: https://index.affectiva.com
generated: '2026-09-12'
method: generated
source: openapi/affectiva-eaas-jobs.json, openapi/affectiva-eaas-entries.json, openapi/affectiva-eaas-representations.json
operations:
  - POST /entries
  - POST /jobs
  - GET /jobs/{jobID}
  - GET /entries/{entryID}
  - GET /entries/{entryID}/representations/{representationID}/media
---

# Analyze a video with Affectiva facial coding

The published contract ships **no operationIds**, so operations are named here by method and
path exactly as they appear in `openapi/affectiva-eaas-jobs.json` and
`openapi/affectiva-eaas-entries.json`. Do not invent identifiers.

## Before you start

- **Auth is HTTP Basic on every call.** Send `Authorization: Basic <base64(user:pass)>`. A
  missing or bad credential returns `401` with body `{"error":"Please login to continue."}`
  and header `WWW-Authenticate: Basic realm="Affectiva Facial Coding API"`. The contract does
  not declare this 401 — treat it as possible on every operation regardless.
- **There is no self-service sign-up.** `https://index.affectiva.com/users/sign_up` returns
  404; credentials come from a commercial agreement.
- **Everything is `https://index.affectiva.com` + the path.** `basePath` is `/` and the
  documents declare no host.

## Steps

1. **Create the entry that will hold the media.**
   `POST /entries` with multipart form data: `entry[media]` (the video file) and
   `entry[storage_type]`. Returns `201`; `422` means model validation failed.
2. **Submit the analysis job.**
   `POST /jobs` with `entry_job[input]` (a file) **or** `entry_job[input_url]` (a string URL),
   plus `entry_job[name]` and `entry_job[storage_type]`. Returns `201`.
3. **Poll for completion.**
   `GET /jobs/{jobID}` until `entry_job[status]` reports the job is done. The API publishes
   **no webhooks and no AsyncAPI**, so polling is the only completion signal available.
   It also publishes **no rate limits** — no `RateLimit-*` header and no `429` anywhere in
   the contract — so poll conservatively and back off on your own schedule.
4. **Read the results.**
   `GET /entries/{entryID}` for the entry, and
   `GET /entries/{entryID}/representations/{representationID}/media` for the media bytes of a
   derived representation. Results are documented as JSON or CSV.

## Rules an agent must follow

- **Retries are not safe on step 1 or 2.** `idempotency.coverage` for this API is `none`
  (see `conventions/affectiva-conventions.yml`): there is no `Idempotency-Key` header and no
  dedupe key on any POST. A retried `POST /jobs` creates a second job and bills a second
  analysis. Confirm with `GET /jobs` before re-sending.
- **There is no cancel and no undo.** The contract publishes no cancel, abort, restore or
  rollback operation, and `DELETE /entries/{entryID}` returns `204` with no stated retention
  window. Do not delete on a user's behalf without explicit confirmation.
- **Do not paginate.** `GET /jobs` and `GET /data_collection_projects` accept no `page`,
  `limit`, `offset` or `cursor` parameter; the whole collection comes back at once.
- **Error handling.** Declared statuses are `400` (invalid integer ID), `404`, `405`, `422`
  (validation) and `500`. The envelope is `{"error": "<message>"}` — not RFC 9457, and there
  is no machine-readable error code. Log the `X-Request-Id` response header; it is
  undocumented but returned on every response and is the only correlation handle available
  to support.
- **Face data is personal data.** The input to this API is video of people's faces. Handle
  consent and retention under the operator's own terms
  (https://www.affectiva.com/terms-of-service/ and
  https://www.affectiva.com/privacy-policy/) before uploading anything.
