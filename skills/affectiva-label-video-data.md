---
name: affectiva-label-video-data
description: Run a human-labeling pass over sampled video frames and segments using the Affectiva EaaS labeling pipeline.
api: Affectiva Facial Coding API
base_url: https://index.affectiva.com
generated: '2026-09-12'
method: generated
source: openapi/affectiva-eaas-frame_sampling_jobs.json, openapi/affectiva-eaas-video_frames.json, openapi/affectiva-eaas-video_segments.json, openapi/affectiva-eaas-labeling_jobs.json, openapi/affectiva-eaas-labeling_tasks.json, openapi/affectiva-eaas-labeling_job_annotations.json
operations:
  - POST /frame_sampling_jobs
  - POST /video_frames
  - POST /video_segments
  - POST /labeling_jobs
  - POST /labeling_tasks
  - GET /labeling_tasks/{id}
  - PUT /labeling_tasks/{id}
  - POST /labeling_job_annotations
  - PATCH /labeling_job_annotations
---

# Label sampled video with the Affectiva labeling pipeline

Four of Affectiva's thirteen published contracts describe a human-labeling pipeline that sits
behind the same HTTP Basic credential as the rest of the API. Operations are named by method
and path because the published documents carry no operationIds.

## Steps

1. **Sample the video.** `POST /frame_sampling_jobs` — "Create a sampling job for video
   frames / video segments". Returns `201`; `422` on validation failure, `500` on server
   error (this operation is one of the eight that declares a 500).
2. **Register the sampled units** if you are supplying them yourself:
   `POST /video_frames` (`201`/`400`/`422`/`500`) and `POST /video_segments`
   (`201`/`422`/`500`).
3. **Open a labeling job.** `POST /labeling_jobs` returns `201`.
4. **Create the tasks.** `POST /labeling_tasks` returns `201`. Read one back with
   `GET /labeling_tasks/{id}` and update it with `PUT /labeling_tasks/{id}`.
5. **Write annotations.** `POST /labeling_job_annotations` creates, and
   `PATCH /labeling_job_annotations` updates. **Note the shape:** these three operations
   carry no `{id}` in the path — they are addressed by the `labeling_job_id` **query
   parameter**, which is unique in this API and easy to get wrong.

## Rules an agent must follow

- **Deletes are terminal.** `DELETE /labeling_jobs/{id}` and `DELETE /labeling_tasks/{id}`
  return `204`, and the provider publishes no restore path and no retention window. Cascade
  behaviour onto child tasks and annotations is **not documented** — do not assume it.
- **No idempotency.** None of these creates takes a dedupe key. A retried
  `POST /labeling_tasks` makes a second task.
- **No schemas.** Not one of the thirteen documents declares a `definitions` block, so the
  request body shape for `labeling_jobs`, `labeling_tasks` and `labeling_job_annotation` is
  not published. Build payloads from a working example obtained from the operator, not from
  the contract.
