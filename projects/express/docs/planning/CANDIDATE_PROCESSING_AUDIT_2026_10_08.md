# Candidate Processing Foundation — Architecture and Code Audit

Status: **AUDIT COMPLETE / IMPLEMENTATION NOT APPROVED**
Date: 2026-10-08
Scope: HigaBase_Plans + higa_systems_express + higa_systems_react.
Review method: repository source and Markdown inspection on GitHub main, **not** execution of local services, live PostgreSQL, Ollama, production integrations, or a new test suite.

## 1. Authority and architectural constraints

Read in this order:
1. HigaBase_Plans/SYSTEM_ARCHITECTURE.md
2. HigaBase_Plans/CANDIDATE_ARCHITECTURE.md (last reviewed 2026-10-08)
3. HigaBase_Plans/AI_CORE.md and ESCO_ARCHITECTURE.md
4. backend PROJECT_RULES.md, CURRENT_TASK.md, PROJECT_STATE.md and domain documents
5. frontend PROJECT_RULES.md, CURRENT_TASK.md, PROJECT_STATE.md and PHOENIX_UI.md

Do not silently turn business SystemUser into Candidate identity. Candidate is a global, independent domain; a Candidate may relate to multiple Organizations, unlike business SystemUser. Each Organization relation requires its own consent/request lifecycle. Resume is distinct from Candidate Profile; an AI draft is neither of them. Optional components link by candidateId instead of adding columns to Candidate. Original source data remains intact; ESCO concepts are independent references, not replacements for text.

Production Candidate-domain code currently requires an explicit repository/service ownership decision: backend PROJECT_RULES.md still declares higa_systems_express business-only, although isolated ResumeDraft research code is present. Do not treat this experiment as an implicit blanket authorization to introduce the production Candidate domain into that repository.

The frontend currently exposes Recruitment -> Candidates for recruiter use. This is a navigation/presentation choice, not ownership of Candidate identity by the optional Recruitment component. Keep its established route until the owner explicitly chooses otherwise.

## 2. Evidence-based implementation inventory

| Layer | Current code evidence | Status | Production limitation |
| --- | --- | --- | --- |
| Named AI task | src/ai/contracts.ts; src/ai/runtime.ts | Implemented | Strict task validation; no durable execution provenance |
| Cloud inference | src/ai/openai-responses-provider.ts | Implemented for parsing | External provider; production choice not finalized |
| Local inference | src/ai/ollama-provider.ts | Implemented for text | No binary file input; no production routing/fallback |
| Parser contract | src/resume-draft/contract.ts; parse-task.ts | ResumeDraftV1 research | Not a final Candidate/Resume schema |
| Draft storage | src/resume-draft/repository.ts; Prisma ResumeDraft | Development research | JSONB + UUID/version/timestamps only |
| Development parse HTTP | src/resume-draft/routes.ts | Text parse + GET by ID | Development-only, no auth/ownership scope |
| Real PDF/DOCX experiment | src/resume-draft/file-parse-cli.ts | Local OpenAI CLI | No production upload endpoint, source store, or queue |
| Benchmark harness | src/ai/benchmark/*; tests/ai/* | Research implemented | Model quality/latency choices still experimental |
| ESCO graph | prisma/schema.prisma; src/esco/import-* | Verified foundation per project docs | No Candidate-linked semantic records |
| ESCO lexical API | src/esco/routes.ts; src/esco/knowledge.ts | GET /api/esco/search | Prefix search only; no occupation/skill proposal workflow |
| Candidate Core + Resume Core | prisma/schema.prisma | Not implemented | No candidateId, profile, Resume source/history model |
| Candidate-Organization relation | prisma/schema.prisma | Not implemented | No consent/relationship ownership boundary |
| Processing job/checkpoints | src/resume-draft/* | Not implemented | No durable intake lifecycle, retries/idempotency |
| Screening / verified facts | Candidate architecture docs | Planned | No session/confirmation persistence |
| Recruiter Candidate list | frontend src/pages/recruitment/Candidates.tsx | UI scaffold | Hardcoded empty rows, no Candidate API; Add modal is informational |
| Candidate-facing intake | central Candidate architecture | Planned | No temporary private consent/screening flow |

README/PROJECT_STATE may lag newer research commits; source code and migrations take priority for observed presence.

## 3. Critical findings and risks

### F-01 — Domain ownership decision is a release blocker (P0)

The central Candidate architecture defines a first-class global Candidate and independent Resume. The backend repository explicitly excludes production Candidate/mobile code without a new approved decision. Settle whether this remains an isolated candidate module in the existing API (explicit exception) or receives a separately owned service/repository. Do not duplicate tables across both. This is a scope/ownership decision, not permission-matrix implementation.

### F-02 — Research draft has no ownership/provenance chain (P0)

ResumeDraft Prisma model stores only id, contractVersion, JSON payload, createdAt and updatedAt. There is no sourceArtifactId, source hash, detected source language, actor, processing request, organization visibility boundary, candidate relation, task/prompt/provider metadata or approval state. ResumeDraftService.parseAndCreate() discards AiRuntime metadata. Good for a parser research artifact; not safe to promote as canonical Candidate/Resume records or share across organizations. Design a provenance-bearing intake record and immutable source artifact reference before production.

### F-03 — Development endpoints are not a production candidate API (P0)

createResumeDraftRoutes exposes POST /api/dev/resume-drafts/parse and GET /api/dev/resume-drafts/:id only when developmentRoutes is true; dependencies.ts sets this from NODE_ENV !== 'production'. There is **no session/ownership check inside these development routes**. The production switch disables them, which is the intended defense, but a dev server exposed to others can allow unscoped draft access. Keep dev API bound to trusted local development; do not deploy it in public staging as a shortcut. Introduce explicit authenticated, scoped, rate-/size-limited endpoints for production.

### F-04 — File ingestion is a CLI experiment (P0)

The CLI reads PDF/DOCX files from a local path, rejects files >= 50 MB, sends base64 input_file to OpenAI, and writes private local JSON results. The general Express JSON limit is 1 MB and the /dev parser accepts documentText only. Thus the frontend cannot currently upload real CV files to a supported production pipeline. SourceArtifact object storage, malware/type validation, reliable extraction, size limits, job processing, retention and deletion policy need a separately approved design. Do not persist originals on application-server local disk.

### F-05 — Missing source facts can collide with parser requirements (P1)

ResumeDraftV1 requires nonempty employer and title for every workHistory item and nonempty institution for education. The processing architecture says missing facts must not be invented. A source with duties but no explicit job title/employer cannot be represented as that workHistory item without a deliberate missing-field policy. The real-PDF reports also recorded empty-field schema failures before the OpenAI schema tightened. Decide source-faithful handling in a versioned research contract, without silently weakening historical ResumeDraftV1 or manufacturing data.

### F-06 — No link between parsed words and ESCO concepts (P1)

ESCO search, concept IDs, occupation-skill projection and multilingual labels exist. There are no Candidate/Resume-to-EscoConcept models, no occupation proposal task and no human confirmation model. ESCO related skills must remain suggestions, never automatically confirmed Candidate capabilities. Start occupation mapping from explicit title + duties + work context; employer name alone must not determine occupation. Retain source text and proposal vs accepted mapping separately.

### F-07 — ResumeDraftV1 is extraction-shaped, not verification-shaped (P1)

Certifications are string[]; dates are raw strings; languages have free-text levels; no provenance per fact, no confidence/uncertainty, no confirmed/rejected state, no original/translated display pair and no inferred-duration semantics. Do not prematurely expand Candidate Core to fit everything. Keep the minimal profile (names, email/phone, languages) separate from Resume history and future candidate capability projections. Source text must allow multilingual characters even though ordinary manually entered business identifiers have ASCII rules.

### F-08 — Identity, consent and cross-Organization access are not optional (P0/P1)

Central Candidate architecture permits one Candidate to have relations with several Organizations, but requires an explicit candidate-confirmed relation before activation. CV intake should avoid revealing whether a Candidate already exists: stable generic recruiter response, safe ambiguous-match handling, no global identity lookup via email. Minimal backend authentication, organization/actor scope, consent token safety, record visibility, retention/deletion and audit events are prerequisites for live candidate data. Only *fine-grained* permissions and full Organization/Location/Department policy matrices remain deferred.

### F-09 — Benchmarks are evidence, not product verification (P1)

Backend CURRENT_TASK.md records, as owner-reported research observations: synthetic local-model comparisons; OpenAI Structured Outputs adjustments; final 14/14 ResumeDraftV1 schema-valid real PDFs with an estimated total API cost of $0.0088234. Schema validity is not extraction accuracy or suitability for production. Keep separate measurements for source completeness, unsupported facts, repeatability, difficult layouts, languages, latency, failure rate and total cost. Do not implement specialist model routing from small fixture counts.

### F-10 — Implementation and documentation drift (P2)

Backend database/state docs list 13 migrations while source tree now contains 20261006170000_resume_draft_research, a 14th migration. Backend README still describes the superseded Organization Company Profile and has no research-module distinction. Frontend PROJECT_STATE predates current candidate navigation; frontend CURRENT_TASK remains VERIFY and must not be marked closed without owner local verification. Reconcile only confirmed current facts; preserve historical verified counts explicitly as historical snapshots.

## 4. Recommended ownership and processing architecture

Conceptual stages (not approved persistence names):

Recruitment -> Add Candidate (upload)
-> intake request (idempotency key + actor/organization scope)
-> immutable SourceArtifact (object key, hash, MIME, retention)
-> document extraction + detected language
-> AI Core candidate.resume.parse@1 (untrusted output)
-> strict ResumeDraft validation + execution provenance
-> candidate identity resolution (private, no existence disclosure)
-> create/reuse global Candidate Core as allowed by identity/consent policy
-> attach source Resume / structured source facts
-> ESCO occupation and skill proposals
-> candidate confirmation + concise screening when required
-> activate Candidate-Organization relation on consent/required screening
-> permitted read model for Recruitment list and Candidate dossier.

SourceArtifact / processing state / ResumeDraft / Candidate / Resume / proposal / screening session / Organization relation are separate responsibilities. Do not turn them into one all-purpose JSON table. Details of retention, pre-consent Candidate creation and when human review is required are design decisions for the next approved block.

Use one durable intake identity for retries and processing. Suggested private process states: RECEIVED, VALIDATING, EXTRACTING, PARSING, NEEDS_REVIEW, WAITING_CONFIRMATION, COMPLETED, FAILED, CANCELED. These are *proposed*, not existing enums. Recruiter display should remain simple: Processing, Waiting for candidate, Needs screening, Ready, with explicit failure handling.

## 5. Prioritized implementation plan (each block independently verifiable)

### Gate 0 — finish/park current research and approve boundary [NEXT]

- Review AI-BENCHMARK-1 evidence; close or explicitly pause it without claiming an unrun model victory.
- Approve the production Candidate owner (same backend isolated module or separate service).
- Decide source-artifact policy, consent point, data-retention basics and minimal allowed actor.
- Record a versioned Candidate Core v1 contract: first/middle/last, email/phone, languages with optional stated proficiency; do not make optional contact values globally unique without an identity policy.
- Resolve absent work-history required-field semantics in research contract before production mapping.
- Acceptance: owner approves the boundary in Markdown; no app/database behavior change.

### Block 1 — Candidate Core v1 + relation isolation

- Append migrations; do not reset existing business or ESCO databases.
- Candidate identity/profile, contact and languages in Candidate-owned records, separate from SystemUser.
- Model Candidate-Organization *request* separate from active, consented relation.
- Scope-safe repository/services with stable error shapes and tests for unauthenticated, wrong-organization, duplicate/ambiguous identity and refusal.
- Acceptance: no cross-organization information leaks and no active relation before confirmation.

### Block 2 — SourceArtifact and durable intake

- Durable artifact ID/checksum/object-storage key; safe PDF/DOCX ingestion/type+size checks.
- Persist intake job lifecycle and idempotency; track source language and failure reason.
- Keep real documents/results outside Git; control retention, PII logging and provider disclosure.
- Acceptance: retry does not create duplicate Candidates; original artifact remains retrievable to authorized processing, and failure does not create canonical facts.

### Block 3 — Candidate Resume Core + parser bridge

- Reuse AiRuntime + versioned ResumeDraft; do not create another AI client stack.
- Translate accepted source facts into Resume-owned history/education/skills/languages with provenance; preserve source values and unknowns.
- Provider/model/version recorded at execution level; never equate AI parse with verification.
- Acceptance: PDF/DOCX -> validated draft -> reviewable canonical source Resume under safe ownership; controlled failure/reparse tests.

### Block 4 — ESCO occupation normalization v1

- Independent proposal/selection objects reference EscoConcept identity, active dataset and source fact.
- Start with experience/occupation; then explicit skills and occupation-suggested skills; confirm separately.
- Use official multilingual ESCO labels; preserve source text and source language.
- Acceptance: deterministic lookup, ranked alternatives/search fallback, no silently confirmed inferred skill, invalid IDs rejected.

### Block 5 — Candidate relation confirmation and screening v1

- Signed/opaque high-entropy one-time links with hashed tokens, expiry and explicit binding.
- Candidate can confirm/refuse relation; one screening engine for candidate/recruiter; confirmation updates one canonical profile, not a copy.
- UI uses derived simple states; record actor/evidence and allow corrections.
- Acceptance: replay/expiry/revoke/other-organization tests; only consented and appropriately screened profiles become Ready.

### Block 6 — Recruitment frontend end-to-end

- Replace empty candidateRows with an authorized, paginated Candidate read model.
- Add Candidate invokes intake entry, with upload and processing status, no mock data.
- Use Phoenix UI structure already approved; keep 28-language i18n and error contracts.
- Acceptance: real happy-path and failure/retry tests, owner visual check, no leakage via search.

### Later, not in basic processing

Advanced permissions matrix, Organization/Department overrides, workflow graph/routing, matching score mathematics, specialist AI router, automatic provider fallback, React Native integration and optional components (housing/planning/etc.).

## 6. Verification matrix for future development

Unit:
- ResumeDraft version and missing-field policies; invalid/partial provider output fails closed.
- multilingual input and source preservation; ESCO graph identity/family filtering.
- duplicate intake idempotency and no hallucinated employer/occupation/dates.

API/integration:
- authentication/session; organization relation scope and consent isolation;
- candidate existence enumeration resistance;
- size/MIME/malformed PDF/DOCX, timeouts, provider unavailable, transient error, reparse;
- lifecycle progress, cancellation/retry and authorized read model;
- token hash/expiry/replay; no relation activation before consent;
- DB migrations on fresh database plus upgrade from existing 14-migration development baseline.

Evaluation:
- real heterogeneous CV corpus: multi-page, tables, language mixing, missing employer/title/institution, duplicate phones/emails, empty contacts;
- structural validation AND fact-level precision/recall, unsupported-fact count, latency/cost;
- no personal CV data, parser output or API keys committed to Git.

Required eventual local owner verification for schema blocks: npm run db:generate, npm run verify, npm run db:status, plus migrations and a controlled end-to-end manual flow. This audit did not execute these commands.

## 7. Explicit decisions requested before code implementation

D1. Production Candidate domain location: isolated module in higa_systems_express as a deliberate rule exception, or separately owned API/repository?
D2. Does a candidate-facing confirmation precede creation of global Candidate identity, or may a restricted pending identity exist first? Ensure identical recruiter-visible response.
D3. Approved artifact storage/retention/deletion solution for original PDF/DOCX and derived drafts.
D4. Minimum legal/business basis for processing an uploaded third-party CV and recruiter access before Candidate consent.
D5. Definition of the first independently verified vertical slice: recommend Candidate Core + safe relation-request skeleton **before** UI upload/AI processing integration.

## 8. Audit outcome and boundaries

**Recommended first action:** Gate 0, followed by Block 1. The risky leap would be wiring the Add Candidate button directly to the unscoped development ResumeDraft endpoints.

No production code was modified by this audit. No authorization expansion, provider selection, new schema or migrations are approved by this document alone. The current AI benchmark task remains IN PROGRESS, and the frontend Phoenix table task remains VERIFY.

References:
- backend: PROJECT_RULES.md, CURRENT_TASK.md, PROJECT_STATE.md, docs/AI_CORE.md, docs/ESCO.md, docs/DATABASE.md;
- source: src/ai/*, src/resume-draft/*, src/esco/*, src/app.ts, src/dependencies.ts, prisma/schema.prisma;
- frontend: src/pages/recruitment/Candidates.tsx, CURRENT_TASK.md;
- cross-project: HigaBase_Plans/SYSTEM_ARCHITECTURE.md, CANDIDATE_ARCHITECTURE.md, AI_CORE.md, ESCO_ARCHITECTURE.md.
