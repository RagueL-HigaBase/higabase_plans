# HIGA Intelligence Orchestration & Monitoring — proposal

Status: **PROPOSED — owner architecture approval required before implementation**
Draft date: **2026-10-17**
Target repositories: higa_systems_express, higa_systems_react, higabase_plans
Scope: multimodal AI execution, traffic admission/quotas, durable queues, live operational monitoring.
Authority: subordinate to AI_CORE.md, SYSTEM_ARCHITECTURE.md, CANDIDATE_ARCHITECTURE.md, AGENT_RULES.md and explicit owner decisions.
This document is a DESIGN PROPOSAL only. No approval for dependencies, DB migrations, routes, UI, service extraction, privacy/authorization changes, provider integration, or rewriting Candidate/ESCO.

**Commercial/metrology companion (2026-10-17):** [HIGA AI Usage, Dynamic Pricing & Billing Engine](HIGA_AI_USAGE_PRICING_BILLING_ENGINE_2026-10-17.md) records owner direction for actual usage-based charging, payer/actor attribution, x per client and task, finance ledgers, monitoring and deferred implementation after Candidate. Commercial agreement does not approve the proposed schema/queues/UI.

## 1. Goal and non-goals

Provide an operational foundation for many concurrent users to execute named AI tasks across text, CV/PDF/DOCX, photos/scans, future audio/video, and structured data. An administrator must see real-time, privacy-safe traffic breakdowns, throughput, queued work, usage and estimated costs, provider failures, and service health.

Core invariants:
- One API key is provider credentials, not a single serial execution channel. Provider/model/project rate and token limits still apply.
- AI is an interpretation service, not domain persistence or authorization authority.
- Preserve original source artifacts; process immutable references and versions; do not embed large binary content in queue payloads.
- Maintain strict typed named tasks and Zod validation; model/provider output is untrusted; domain services own canonical writes.
- Separate technical execution records from Candidate, Organization, ESCO, and other domain aggregates.
- Keep client sessions, user data, organization scope, and provider secrets separated.
- Begin as a modular monolith + independent workers, not separate databases/microservices per modality.

Out of scope for this architecture-only block: executable code, production rollout, final model selection, permanent pricing rates, full RBAC policy, video processing choices, and historical production SLA claims.

## 2. Source-inspected baseline (GitHub main; not a local runtime verification)

Express package.json currently lists Express, Prisma/PostgreSQL, Pino, Zod; no Redis/BullMQ dependency.
Existing files:
- src/ai/contracts.ts — named task/version contracts.
- src/ai/runtime.ts — validates input, invokes provider, validates output, returns duration/provider/model/token metadata.
- src/ai/provider.ts and src/ai/openai-responses-provider.ts — provider boundary; Responses API implementation for CV interpretation.
- src/dependencies.ts — builds AI runtime and ResumeDraftService dependency directly.
- src/resume-draft/service.ts — direct await of AiRuntime.execute for file preview and parsing; no durable execution queue in this path.
- src/resume-draft/object-storage.ts — S3-compatible stored file read/write/delete.
- src/resume-draft/routes.ts — development-only CV source inspection and upload routes.
- src/app.ts — existing business HTTP API.

Thus, reuse AI Core and object storage. Do not assume current implementation already has resilient queue, tenant quota, persisted provider-call cost ledger, or realtime admin monitoring.
Do not modify current Candidate V2, ESCO, permissions, or CV prototype in this proposal.

## 3. Proposed component boundaries

Client (React / Native / API integration)
  -> Existing Express authentication, route-level authorization, admission control
  -> AI Task API / Media upload service (validated task ID, identity, domain scope, file ref)
  -> AiJob persistent intent (PostgreSQL) + transactional outbox
  -> Redis + BullMQ queues: interactive, batch, media
  -> Worker pool (separate Node processes/containers; horizontally scalable)
  -> Existing typed AiRuntime and provider adapters (OpenAI/Ollama/future)
  -> Task output schema and domain-service checkpoint
  -> Technical AiJob/AiProviderCall ledger and immutable source/derived artifact references
  -> Aggregated metrics/events stream to authenticated admin UI

The job store is authoritative for business-visible status. Redis/BullMQ is execution coordination, not authoritative canonical domain data. Use transactional outbox (or explicitly audited reconciliation) to avoid DB-committed jobs that were never enqueued and vice versa.

## 4. Multimodal task pipeline

Define a transport-independent task envelope:
- jobId; taskName; taskVersion; inputContractVersion; modalitySet; inputArtifactRefs; sourceChecksum/version;
- actorSystemUserId (or future appropriate principal), access scope, organizationId only where authorized by owning domain;
- originatingDomain/entityRef; idempotencyKey; correlationId; requestedAt; deadline; priorityClass;
- provider/model policy identifier, processing/prompt version, allowed tools and cost budget;
- no file binaries, secret tokens, raw CV contents, or candidate PII in Redis job data.

Input categories are orthogonal to tasks:
- TEXT: chat/message, simple extraction/classification.
- DOCUMENT: PDF, DOCX and later supported document types; can contain text and embedded scans.
- IMAGE_INPUT: photo, screenshot, scan, certificate (vision / OCR / inspection).
- IMAGE_OUTPUT: generated/edited images; separate usage/cost semantics.
- AUDIO_INPUT/AUDIO_OUTPUT: voice recordings, transcription, synthesis.
- VIDEO: bounded segment/frame/audio preprocessing, workflow plus downstream model operations.
- STRUCTURED: JSON, records, event payloads.

One document might lead to several dependent steps: validate file -> preserve immutable source -> extract text or sample pages -> run model -> validate output -> produce draft -> review. Each step has a typed status and measures processing and AI-provider calls independently.

Artifacts:
- Use the existing S3/MinIO object-storage mechanism initially.
- Validate real magic bytes, MIME allowlist, file size, image dimensions and media duration; scan potentially malicious uploads; isolate decompression/decoder workers.
- Preserve source checksum, size, derived checksum, media dimensions/pages/duration, owner/reference and retention class.
- Avoid passing raw files as base64 through frontend API routes at production scale; prefer controlled direct-to-object-storage upload with short-lived scope-limited upload authorization where approved.
- Media extraction and thumbnails must not silently mutate originals. Remove unneeded EXIF/geolocation from derived previews; retain source only under defined retention/privacy policy.

## 5. Scheduling, fairness and limits

Separate worker/queue capacity from provider budgets:
- Queue lanes: interactive (fast UX), batch (CV collections/reprocess), media (CPU/GPU/conversion).
- Apply actor, business organization, task type, upload bandwidth/bytes, and provider/project/model quotas at API admission and in workers.
- Global distributed RPM/TPM and concurrency budgeting per provider/project/model, including estimated prompt/output and image-equivalent tokens where applicable.
- Pre-admission reserves budget, post-response reconciles actual usage and releases unused reservation. Missing provider usage must be marked unknown/estimated, never exact.
- Fair sharing: per-organization caps and queue rotation/weights so one large tenant cannot starve other tenants. Standard BullMQ concurrency/rate limiting alone does not provide a complete per-tenant fairness guarantee; implement/test explicitly or evaluate group-support licensing before choosing it.
- A bounded policy selects provider/model capabilities; fallback only if the task privacy/security/data residency and output contract permit it.
- On 429 and transient provider failures: bounded exponential backoff with jitter, Retry-After when provided, circuit breaker; configurable retry ceilings and dead-letter review.
- Input/schema/domain validation failures are not automatically retried.
- Idempotency for submitted jobs and side effects; an interrupted provider request may have been billable even if response was lost. Do not promise exactly-once provider billing.
- Cancellation is best-effort when already running: prevent future steps; preserve audit records; cancellation cannot guarantee refund/undo of an upstream call.
- Optional later priority aging and backlog-based autoscaling.

Do not hardcode concurrency=10 or 100 as product limits: calibrate to actual provider quota, latency, payload size, model cost, CPU/memory and burst testing.

## 6. Technical data ownership and suggested schema (names provisional)

New isolated AI-operations models in existing PostgreSQL, no duplicate Candidate/Organization business tables:

AiJob:
  id; taskName; contractVersion; status; priority; actorId; domainRef; authorizedScopeRef;
  idempotencyKey; inputArtifactRef(s); sourceVersion/checksum; createdAt; startedAt; endedAt;
  deadlineAt; processingVersion; requestedProviderPolicy; cancellationRequestedAt; errorCode.

AiJobStep:
  id; jobId; stepType; status; attempt; workerId; startedAt; finishedAt; metricsJson (sanitized and bounded);
  resultArtifactRef; errorClass.

AiProviderCall:
  id; jobId; stepId; provider; model; providerProjectTag; requestId;
  startedAt; endedAt; statusCode; retryIndex; inputTokens; outputTokens;
  modalityUsageJson; estimatedCost; pricingCatalogVersion; currency; costIsEstimate; sanitizedErrorCode.

AiJobEvent:
  id; jobId; timestamp; eventType; sanitizedPayload; actorId (where relevant).

AiOutbox:
  id; aggregateId/jobId; eventType; dispatchStatus; attempts; createdAt.

AiMetricBucket (optional at first):
  bucketStart; scope; taskType; inputModality; provider; counts; bytes; calls; tokens; estimatedCost;
  aggregation updates batched, not per API event full-table rewrites.

Consider indexed time-series retention/partitioning only when warranted. Avoid storing raw prompts, CV contents, full file paths, sensitive image thumbnails or arbitrary unredacted model output in telemetry. Correlation IDs connect actions; logs, events and billing records have different retention/access policies.

## 7. Three distinct types of traffic

A. Network/API traffic:
- HTTP requests/second; response codes; uploaded/downloaded bytes; MIME; ingress/egress bandwidth;
- user sessions or distinct actors as appropriate (not equal to active AI tasks);
- endpoint latency p50/p95, in-flight HTTP requests.

B. Workflow traffic:
- submitted/running/completed/failed/cancelled/retrying AI jobs;
- queue backlog and oldest waiting age; worker utilization; step durations; time-to-result;
- taskName, modality, source type, owning domain, provider policy, tenant scope.

C. AI provider usage/cost:
- actual provider API calls and 429/5xx rates; model/provider; RPM/TPM estimates versus known caps;
- input/output tokens when reported; image generation count/quality/resolution; audio minutes where reported;
- estimated cost from versioned rate catalog, budget remaining, reconciliation status;
- explicitly distinguish provider-billed usage from internal bytes and from user-job counts.

A single uploaded PDF can create one HTTP upload, one durable AI job, several preprocessing steps and multiple provider calls. Never merge these counters into one 'request' number.

## 8. Real-time Phoenix administration UI (proposed, not implemented)

Route proposal: /system/ai-operations (exact route and RBAC require owner approval).

Overview, refreshed by authenticated SSE stream of sanitized aggregates:
- cards: API RPS, network inbound/outbound MB/s, queued jobs, in-flight jobs, p95 latency, estimated AI spend;
- 1m / 5m / 1h / 24h time-series of API requests, queue growth, success/error, tokens/min and bytes/s;
- modality breakdown: text/document/image-input/image-output/audio/video/structured;
- task breakdown: resume.parse, future image.inspect, message.classify etc.;
- provider health: provider/model, in-flight, recent 429/5xx, retry rate, concurrency/quota saturation;
- jobs table: jobId, authorized domain association, task, modality, sanitized status/error, user/tenant scoped for viewer;
- budgets/alerts: daily spend threshold, queue age, error-rate spike, media-processing saturation, provider exhaustion;
- filters: time range, modality, task name, provider/model, relevant organization scope.

Role/scoping baseline:
- Platform admin/owner: platform-wide operational aggregates only with deliberate disclosure controls; no default raw source files/prompts.
- Organization admin: only authorized organization-specific aggregate rows; cannot query other tenants.
- Regular actor: only their authorized job status/results.
- Current Candidate domain is global and may have zero or multiple organization relationships; NEVER infer Candidate authorization from one organizationId field in a technical job. Scope checks delegate to domain authorization policy.

SSE for one-way updates is enough for version one. Initial server-side aggregation cadence about 2-5 seconds, with graceful reconnect, heartbeats, backlog-safe event IDs and bounded per-connection resources. History snapshots via paginated authenticated REST. WebSockets only if justified by bidirectional interaction.

Actions (pause lane, throttle, cancel, retry) deferred behind explicit admin approval, irreversible-side-effect protection and full actor audit.

## 9. Observability and storage strategy

Instrumentation: structured Pino events and correlation/trace IDs; OpenTelemetry when implemented for HTTP, queue, provider calls and storage. Never attach sensitive request bodies as span attributes. Consider Prometheus/Grafana/Loki only as operational need grows; initial PostgreSQL aggregates + SSE admin page can be sufficient for development.

Cost accounting tracks per provider operation with actual usage fields where available and versioned tariff snapshots; displayed USD/EUR figures remain estimated until verified against provider billing. No made-up historical zeroes. Media cost units may not be tokens. Avoid unbounded labels in metrics systems (file IDs, emails, candidate IDs, or full URLs); keep high-cardinality detail in access-controlled DB event records.

Retention, GDPR lawful basis, purpose limitation, EU data controls/provider DPA, log minimization, data deletion and audit retention require an explicit production policy, especially for CVs, personal photos, candidate audio/video, and inferred personal attributes.

## 10. Preliminary integration points

Backend:
- keep src/ai/contracts.ts, runtime.ts, provider.ts and existing Zod schema gates;
- introduce proposed src/ai-orchestration/{task-registry,job-service,admission,queues,worker,usage,telemetry}/ modules;
- add separated worker process entrypoint and dependency injection, worker health and graceful shutdown;
- proposed authenticated APIs for job submission/status/cancel and scoped monitoring read/stream;
- extend provider adapters with timeouts, robust error categories and usage normalized by modality;
- add approved Prisma technical entities and migrations separately; preserve existing Candidate schema.

Frontend:
- proposed Phoenix System Admin AI Operations page, scope-aware server aggregates, REST initial load + SSE live deltas;
- job state/progress UI in owning domain screens, no raw technical errors and no default exposure of other users' tasks.

Infrastructure:
- existing PostgreSQL and S3/MinIO;
- Redis/BullMQ added only after approval and dependency/security review;
- separate queue worker service in Docker Compose and later deployment scaling;
- provider API keys only in server/worker secret environment; never browser or job payload.

## 11. Staged delivery with explicit approvals (not automatic implementation)

Block A — Observability contract and baseline:
  define task/modality/usage taxonomy; instrument AiRuntime provider call outcomes and correlation IDs;
  collect sanitized execution metadata; no dashboard or new file-flow changes until contract approved.
  Verify unit tests for privacy filtering, token usage, metadata correctness, current parser parity.

Block B — Durable jobs:
  DB job/step/outbox schema, Redis/BullMQ, queue worker and idempotent one-task pilot
  (candidate.resume.parse, WITHOUT replacing current dev paths until verified).
  Verify 100 simultaneous submissions, crash/restart, duplicate requests, 429 backoff, rollback and data isolation.
  Requires migration/deployment and scope approval.

Block C — Live AI Operations:
  aggregate REST/SSE APIs, role/scope checks, React Phoenix admin dashboard;
  historical filters, explicit estimated cost, worker and provider statuses.
  Verify correct totals under burst/reconnect and no leakage of PII or unauthorized tenant traffic.

Block D — Images/media input:
  validated image/scanned-document input, media-preprocessing workers, provider capability routing,
  storage retention/security and separate image cost units. Verify malicious/oversized files and output policy.

Block E — Audio/video and scale:
  separate transcription/video pipelines, bounded extraction and additional autoscaling/observability only
  when supported by real product use cases and privacy policies.

Do not start implementation blocks, edit canonical architecture, run migrations, or alter production routes without separate owner approval of exact scope.

## 12. Acceptance scenarios

1. 100 concurrent users submit mixed tasks: correct auth, per-tenant isolation, bounded queues; no accidental cross-user results.
2. One tenant submits 1,000 bulk CVs while others chat: interactive tasks have service under approved fairness policy.
3. Provider returns 429 or becomes unavailable: retry and circuit break behave as configured; no silent job loss.
4. Worker process crashes between DB commit and enqueue/provider completion: outbox repair and idempotency prevent lost workflows/duplicate canonical writes.
5. PDF includes embedded scans; photos have metadata; audio/video has unsupported codecs: source preserved, supported transformations and rejection recorded.
6. Provider bills for a timed-out request: usage marked unknown/estimate and reconciled, not treated as zero.
7. Admin sees charted upload bytes, modality, backlog, tokens, retry errors and estimated spend; unauthorized viewer sees no other tenant's data.
8. Agent reprocesses updated CV using same or new source artifact: versioned result is new, original and human-confirmed Candidate facts stay unchanged.
9. Admin connection is interrupted: SSE reconnects, complete history remains queryable via REST, counters do not double-count events.
10. Privacy test: API responses, logs, job queue payloads, traces, telemetry and dashboard omit raw CV, image content, credentials and PII.

## 13. Owner decision gate

Approve or revise, separately:
A. modular monolith + separate worker process + PostgreSQL/S3/Redis architecture;
B. Redis/BullMQ choice and operational deployment;
C. technical AI job/call/outbox schema and migration ownership;
D. tenant/actor fairness, provider quota and budget policies;
E. privacy, file/video retention, provider routing/data residency, authorization and admin visibility;
F. dashboard route/metrics and Phoenix design provenance;
G. narrow first implementation block.

Until then this remains PROPOSED. No runtime verification, DB modification, local Git state assessment or owner visual acceptance has occurred.
