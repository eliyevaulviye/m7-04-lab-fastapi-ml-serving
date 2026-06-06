# API Design Decisions

## Versioning

Path-based versioning (`/v1/`) was chosen because it is unambiguous in logs, gateway
routing rules, and browser history — any proxy or load balancer can route by prefix
without inspecting headers. Header-based versioning (`Accept: application/vnd.visionmod.v1+json`)
is harder to discover, invisible in plain URLs, and breaks caching in most CDN configurations.

## Batch ordering and partial failures

When `predict-batch` receives 32 items and one image is corrupt, the response is HTTP 200
with a `results` array that contains one entry per submitted item keyed by the caller-supplied
`id`; the corrupt item carries `"status": "error"` with a machine-readable `code` and
human-readable `message`, while all other items carry `"status": "ok"` with their labels.
This "partial success" model was chosen over failing the whole batch because callers
have already paid network cost for all 32 items, and a single bad upload should not force
a retry of 31 valid ones. Keying results by caller-supplied `id` rather than array position
means clients can process results safely even if they submitted items concurrently.

## Async lifecycle

A job moves through four states: `queued` (accepted, not yet dispatched to a GPU worker),
`running` (inference in progress), `done` (labels available), and `failed` (inference error
or timeout after 60 s). Results — whether `done` or `failed` — are retained for 24 hours
from job creation (the `expires_at` field in the response), after which the record is purged
and `GET /v1/predictions/{job_id}` returns 404. Twenty-four hours gives callers enough time
to collect results across timezones and business-hours batch jobs, without accumulating
unbounded storage for a multi-tenant service.
