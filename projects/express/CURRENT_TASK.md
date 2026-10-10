# Higa Systems Express — Current Task

Status: **IN PROGRESS**

## AI-BENCHMARK-1 — reproducible local CV parser benchmark

Goal:
- build a repeatable benchmark harness around the existing `candidate.resume.parse@1` contract;
- compare local models using the same source CV text, task instructions, strict ResumeDraftV1 validation and measurement rules;
- choose Higa's local CV parsing engine from evidence rather than generic model rankings.

Initial model set:
- `gpt-oss:20b` — verified baseline;
- `qwen3.5:9b` — smaller candidate;
- `gemma4:12b` — medium candidate.

Benchmark dimensions:
- strict contract pass/fail;
- JSON/provider failure;
- latency;
- provider-reported input/output token counts when available;
- extraction completeness against curated expected facts;
- unsupported/invented facts (hallucination);
- repeatability across repeated runs.

Implementation boundary:
- benchmark tooling is development/research only;
- no production request routing or automatic provider fallback;
- no Candidate production persistence changes;
- no ResumeDraftV1 weakening;
- no ESCO scoring in this first benchmark;
- no PDF/DOCX extraction in this first benchmark;
- benchmark source CVs and expected facts must not contain secrets/API keys.

Method:
1. establish a small curated fixture set with explicit expected facts;
2. run every model against exactly the same fixtures and task contract;
3. record machine-readable results;
4. repeat enough runs to expose instability and warm/cold latency;
5. expand to heterogeneous real CVs only after the harness itself is verified.

Last closed block: **OLLAMA-PROVIDER-1 — VERIFIED**


Implemented:
- development-only `npm run ai:benchmark:resume` CLI;
- model list configurable through `BENCHMARK_MODELS`;
- repetitions configurable through `BENCHMARK_REPETITIONS`;
- same `candidate.resume.parse@1` + strict ResumeDraftV1 for every model;
- deterministic expected-vs-actual fact scoring (precision, recall, F1, unexpected facts);
- latency and Ollama token counts recorded when available;
- machine-readable JSON report written locally;
- initial synthetic English recruiter fixture;
- scoring unit tests;
- no database writes and no production routing changes.

Verification required:
1. owner `git pull`;
2. `npm run verify`;
3. `npm run ai:benchmark:resume` with local Ollama and both installed models;
4. inspect the generated report before expanding fixtures or repetitions.


## Benchmark findings — 2026-10-07

Tested so far:
- local Ollama models: `gpt-oss:20b`, `qwen3.5:9b`, `gemma4:12b`;
- the same 10 synthetic CV fixtures and the same strict `ResumeDraftV1` contract;
- repeated runs for contract reliability, latency, extraction F1, missing/unexpected fields and prompt sensitivity;
- a neutral Round-1-style parser prompt plus an explicit source-only summary rule;
- an experimental stricter Round-2 prompt, which was rejected because it caused model-specific regressions, especially in name parsing and Qwen extraction quality.

Observed direction:
- models have different error profiles rather than one model being uniformly best;
- `gpt-oss:20b` has shown the strongest overall extraction quality but higher latency and occasional provider/output-length failures;
- `qwen3.5:9b` has shown the strongest speed and contract reliability, with recurring headline/name omissions;
- `gemma4:12b` can be highly accurate on successful outputs but currently has weaker strict-contract reliability, especially around optional location output;
- a monolithic whole-resume F1 is not enough to choose future model routing.

Next research direction:
- measure micro-precision/recall/F1 per semantic resume block without allowing absent blocks to inflate scores;
- record source coverage plus expected/matched/unexpected facts for each block;
- use those measurements to test the hypothesis that different models should act as specialists for identity, contacts, experience, skills/tools, education/certifications, summary/languages/achievements;
- keep specialization as benchmark evidence first: do not add production routing or multi-model orchestration until the measurements justify it;
- future Candidate Intake (typed/spoken free-form information when no CV exists) remains a separate task family from document parsing, with speech-to-text outside the resume parser.


## Targeted hard-fixture phase — 2026-10-07

Implemented next:
- added five harder synthetic fixtures focused on work history, skills/tools separation, education/certifications, multilingual languages, and summary/achievements;
- added exact-id fixture filtering through `BENCHMARK_FIXTURES` so targeted experiments do not require a full suite run;
- unknown fixture ids fail explicitly instead of being silently ignored;
- targeted fixtures remain synthetic and contain no real candidate PII.

Reason:
- the first block-level micro-F1 run showed useful differences for identity, contacts and summary;
- languages, work history, skills, tools, education and certifications were still too easy and produced artificial-looking ties at 1.000;
- the next benchmark phase should increase task difficulty rather than merely increase repetitions.

Execution policy for this phase:
- use one repetition for exploratory specialist comparisons;
- use fixture filtering for focused experiments;
- reserve repeated/full-suite runs for validation of a candidate routing decision;
- do not implement production multi-model routing until harder block-level evidence supports it.

## Capability fixture coverage phase — 2026-10-07

Evidence from the first five hard fixtures:
- all three models passed the strict contract for all five runs;
- overall mean F1: `gpt-oss:20b=0.971`, `qwen3.5:9b=0.914`, `gemma4:12b=0.902`;
- GPT-OSS remained the cleanest overall extractor but was materially slower;
- Qwen matched GPT-OSS on several target blocks while running much faster, but showed a strong certification weakness;
- Gemma reached full recall in several blocks but showed repeated over-extraction in contacts, skills and achievements;
- summary extraction remained unresolved: all three models scored block F1 `0.000` on the hard summary fixture;
- most specialist block conclusions were still based on only one source fixture (`src:1`), so routing decisions are not yet justified.

Implemented for the next phase:
- added 18 small capability fixtures, three each for:
  - work-history promotion / overlap / missing-date handling;
  - mixed skills-vs-tools separation;
  - language extraction without levels, incidental language mentions and mixed level wording;
  - education-vs-certification boundaries, incomplete education entries and certification date noise;
  - summary-vs-achievement boundaries;
  - multiple contacts, location noise and no-location cases;
- kept `BENCHMARK_REPETITIONS=1` for exploratory runs;
- kept scoring and ResumeDraftV1 unchanged so new results stay comparable with earlier runs;
- added a unit-test guard requiring at least four independent source fixtures for every targeted semantic block when the hard + capability suite is considered together.

Recommended execution:
1. `git pull`
2. `npm run verify`
3. run one capability group at a time with the existing exact-id `BENCHMARK_FIXTURES` filter;
4. compare block micro-F1, extras and latency before considering any production routing.

Do not implement specialist production routing yet. The current goal is to establish repeated semantic evidence for each block, not to optimize the runtime architecture prematurely.

## Certification contract note — 2026-10-07

Targeted education/certification benchmark result:
- Education: `gpt-oss:20b` achieved `16/16` expected facts with no extras; Qwen kept full recall but added two facts; Gemma lost education facts and added extras.
- Certifications: all three models produced the same apparent `0.778` micro-F1 because `ResumeDraftV1.certifications` is currently `string[]`.
- The shared misses were semantic matches with date/status text preserved, e.g. expected `VCA VOL` vs returned `VCA VOL valid until 2028`, and expected `NEN 3140 VP` vs returned `NEN 3140 VP completed 2025`.
- Treat this as a contract-modeling limitation rather than evidence that all three models failed to recognize the certifications.
- Do not change the contract during the current benchmark phase. Revisit certification structure later (name + lifecycle/date metadata) after the remaining semantic blocks are measured.

## Summary / achievements benchmark note — 2026-10-07

Targeted four-fixture result:
- Achievements are currently closed for this benchmark phase: all three models matched all 9 expected achievement facts with 0 extras (micro-F1 `1.000`).
- Summary extraction remains unresolved:
  - `gpt-oss:20b`: `1/4` expected summaries matched, micro-F1 `0.400`;
  - `qwen3.5:9b`: `0/4`, micro-F1 `0.000`;
  - `gemma4:12b`: `1/4`, micro-F1 `0.400`.
- None of the three models added unsupported summary facts in this targeted run; the dominant failure mode was omission of explicit summary text.
- Keep the current parser contract unchanged for now. Revisit summary extraction as a separate prompt/contract issue after the remaining capability blocks are measured.

## External OpenAI baseline phase — 2026-10-07

Purpose:
- compare the same `candidate.resume.parse@1` benchmark contract against an external low-cost production candidate;
- keep local models as development/research baselines while measuring whether an API-first Production V1 is simpler and cheaper than operating dedicated GPU infrastructure.

Implementation:
- the existing `OpenAiResponsesProvider` is reused; no separate resume architecture is introduced;
- `BENCHMARK_OPENAI_MODELS` adds OpenAI targets beside the existing Ollama `BENCHMARK_MODELS`;
- an OpenAI-only run is possible by setting `BENCHMARK_MODELS` to an empty string;
- benchmark OpenAI runs explicitly use `reasoning.effort=none` to match the current Ollama `think:false` extraction mode;
- measured input/output token usage is recorded and converted to an estimated USD cost for known Luna model ids;
- the API key remains environment-only and is never written to benchmark reports.

Initial external baseline:
- `gpt-6-luna`;
- current benchmark pricing baseline: USD 0.10 / 1M input tokens and USD 0.50 / 1M output tokens;
- `gpt-5.6-luna` pricing remains available for historical comparison.

Scope boundary:
- this phase changes benchmark orchestration only;
- it does not select a production provider, change ResumeDraftV1, alter ESCO/SKA logic, or add automatic local/cloud fallback;
- production routing remains an evidence-based decision after the external benchmark results are available.

## GPT-6 Luna initial full-suite result — 2026-10-07

Initial external run:
- provider/model: `openai:gpt-6-luna`;
- fixtures: 33;
- strict validation pass rate: `21/33 = 0.636`;
- validated-pass mean F1: `0.964`;
- validated-pass median F1: `1.000`;
- mean latency on validated passes: `1916 ms`;
- median latency on validated passes: `1660 ms`;
- recorded successful-run cost: `$0.001636` total / `$0.000078` mean per successful synthetic fixture.

Interpretation:
- all 12 validation failures were caused by the optional `summary` field being emitted as an empty or non-string missing value (`too_small` / `invalid_type`);
- this is a provider response-shape mismatch, not evidence of poor CV understanding;
- on validated runs Luna achieved perfect target micro-F1 for identity, languages, work history, skills, education and achievements;
- certification mismatch remains affected by the already-recorded `string[]` certification contract limitation.

Follow-up:
- use OpenAI Structured Outputs with a strict provider-specific schema;
- represent missing optional scalar fields as `null` externally, then strip null object fields before the shared ResumeDraftV1 validator;
- do not change ResumeDraftV1 semantics or the Ollama benchmark contract.

## Real CV file ingestion phase — 2026-10-07

Purpose:
- move beyond synthetic text fixtures and test actual candidate CV files end to end;
- keep real candidate documents and parsed output strictly local and outside Git.

Implementation:
- `candidate.resume.parse@1` now accepts either raw `documentText` or a provider-side `documentFile` input;
- supported real-file inputs for this phase: PDF and DOCX;
- OpenAI Responses receives files inline as base64 `input_file` content, so no persistent Files API upload is required;
- PDF uses `detail=auto` so the model receives extracted text plus page visual context; DOCX is processed as document text by the Responses API;
- the existing strict Structured Output / ResumeDraftV1 validation remains unchanged;
- Ollama remains text-only and rejects binary file input explicitly;
- added `npm run ai:parse:resume-files -- <file-or-directory>`;
- directory runs are sequential and save full parsed drafts only to ignored local `benchmark-local/real-cv-results.json`;
- console output contains technical metrics and counts, not parsed PII values;
- the CLI records latency, token usage and estimated OpenAI cost for each CV.

Privacy boundary:
- `data/` and `benchmark-local/` are already ignored by Git;
- real CV files must never be committed;
- parsed real-CV reports must never be committed;
- `OPENAI_API_KEY` remains environment-only;
- provider requests use `store:false`.

Initial execution:
`npm run ai:parse:resume-files -- .\\data\\benchmark-cv`

This phase does not write Candidate/ResumeDraft records to PostgreSQL and does not yet introduce frontend upload.

## Real CV first run — 2026-10-07

Observed on 14 local PDF CVs with `gpt-6-luna`:
- `12/14` passed the shared ResumeDraftV1 validator;
- successful-run estimated cost: `$0.007360` total;
- the two failures were provider-shape mismatches only:
  - empty `education[].institution`;
  - empty `workHistory[].title`.
- align the OpenAI strict JSON schema string bounds with ResumeDraftV1 so required strings cannot be empty and optional nullable strings cannot be empty when present.
- no change to ResumeDraftV1 semantics, Candidate persistence, ESCO/SKA, or frontend.

## Real CV second run — 2026-10-07

Observed after aligning OpenAI string length bounds:
- `13/14` real PDF CVs passed ResumeDraftV1 validation;
- successful-run estimated cost: `$0.007665` total;
- the previous empty `workHistory[].title` failure disappeared;
- one CV still failed because two `education[].institution` values became empty after Zod trimming;
- likely cause: whitespace-only strings satisfy JSON Schema length constraints but fail the shared `.trim().min(1)` contract;
- follow-up: add a non-whitespace `pattern` constraint to OpenAI string fields so provider output cannot use whitespace-only placeholders.

No ResumeDraftV1 semantic change is required.

## Candidate screening architecture foundation — 2026-10-07

This section records the agreed product/architecture baseline after the first real-CV validation round.

### Real-CV parser validation

Final real-PDF run with `gpt-6-luna`:
- `14/14` real PDF CVs passed `ResumeDraftV1` validation;
- total estimated cost: `$0.0088234`;
- mean estimated cost: about `$0.000630` per CV;
- real CVs confirmed that the remaining product challenge is no longer basic document parsing, but normalization, verification and screening UX.

### Core principle

Keep the user experience simple even if the backend is more sophisticated.

The canonical flow is:

```text
CV / candidate input
        ↓
Luna extracts source facts
        ↓
ESCO occupation normalization
        ↓
ESCO-related skill suggestions
        ↓
candidate OR recruiter screening
        ↓
verified candidate capability profile
        ↓
future SKA / matching
```

The parser extracts facts. ESCO normalizes and proposes. A person confirms reality.

### Source facts must remain separate from normalized meaning

Never overwrite or fabricate source facts just to satisfy a normalized profile.

Examples:
- if employer is missing in the CV, keep employer absent;
- if job title is missing, keep source title absent;
- if an employer is present, do not infer the occupation from the employer name;
- original descriptions remain preserved as source evidence.

Occupation normalization should be driven primarily by:
1. explicit source job title when present;
2. duties / description;
3. tools and work context;
4. employer only as weak contextual evidence.

A company name such as DHL must never imply one fixed occupation by itself.

### ESCO occupation selection

The system should propose one primary normalized ESCO occupation for a work-history item or relevant candidate capability block.

If the proposed occupation is wrong:
- show a small ranked set of nearby/related ESCO occupations;
- prefer a compact set such as 5–8 useful alternatives rather than exposing the full taxonomy;
- allow a search fallback when the correct occupation is not in the initial suggestions;
- do not make product behavior depend on a fixed visible graph-hop count. Graph distance is an internal ranking mechanism.

The recruiter/candidate should see a simple occupation label, not graph internals, relation metadata or confidence mechanics.

### Skills: explicit facts vs ESCO suggestions

The screening pool is the useful maximum, not the final truth.

Keep two logical groups:
- skills explicitly supported by the CV / candidate input;
- skills suggested from the selected ESCO occupation and its relevant occupation-skill relations.

ESCO-suggested skills do not become confirmed candidate skills automatically.

The interaction should be primarily:
```text
Skill?
[ Yes ] [ No ]
```

Manual skill search/add is a fallback, not the normal workflow.

Do not expose dozens of ESCO skills at once. Prioritize a short screening set:
1. explicit CV skills that matter;
2. essential ESCO occupation skills;
3. highly relevant optional/related skills;
4. omit low-value tail items unless requested.

### One screening engine, two actors

There must be one screening model used through two interfaces:

```text
                 Screening session
                       │
             ┌─────────┴─────────┐
             ↓                   ↓
       Candidate link        Recruiter UI
       browser/mobile        during phone call
```

Candidate path:
- recruiter/system creates a private temporary screening link;
- link may be delivered by email or WhatsApp;
- no app installation should be required;
- account creation should not be required for the basic verification flow;
- the candidate answers the same questions the recruiter would answer during a call.

Recruiter path:
- if the candidate did not complete self-screening, the recruiter opens the same screening flow during the phone call;
- recruiter records the candidate's answers directly;
- both paths must produce the same backend result.

### Screening UX rule

Do not build a long questionnaire page.

Preferred interaction:
- one question at a time, or very small groups;
- mostly Yes / No confirmation;
- occupation confirmation first when needed;
- selecting another occupation refreshes the relevant skill questions;
- candidate/recruiter can add a missing occupation or skill only when necessary.

The target is a fast verification flow, not a detailed interview form.

### Minimal screening status

Keep status intentionally small:

```text
NOT_SCREENED
IN_PROGRESS
SCREENED
```

Saving partial answers does not imply completion.

A profile becomes `SCREENED` only after the required screening path is completed.

The backend may record whether completion was performed by the candidate or recruiter, but this should not complicate the recruiter-facing workflow.

### Private screening link

Future implementation should use a temporary private token-based link.

Expected security shape:
- random high-entropy token;
- server stores only token hash;
- candidate/profile/request binding;
- expiration;
- single screening request state;
- explicit revoke/replace capability when later required.

Delivery channels can later include:
- copy link;
- email;
- WhatsApp Business integration.

### Multilingual behavior

Original free text must never be destroyed.

For descriptions, summaries and other free text:
- preserve original text and detected/source language;
- provide translated display text according to the active system language;
- translations should be generated/cached on demand rather than pre-generating every supported language.

For ESCO occupations and skills:
- prefer official ESCO multilingual labels from the local ESCO graph;
- do not use AI translation when an appropriate official label exists.

The user should normally see translated content automatically, with an optional "view original" affordance.

### Matching boundary

Future matching must not score directly from dirty/raw CV wording.

Preferred matching inputs:
- normalized ESCO occupations;
- confirmed skills/capabilities;
- verified languages/certifications and other structured profile facts;
- source evidence remains available for audit/display but is not the primary matching representation.

The exact matching mathematics, graph-distance weighting and scoring percentages remain a separate future design block.

### Future self-service intake

The same screening engine should support a later public/self-service flow:

```text
candidate opens vacancy or intake page
        ↓
uploads CV
        ↓
CV parsed automatically
        ↓
occupation/skills proposed
        ↓
candidate completes fast screening
        ↓
verified profile is created/updated
```

This does not require React Native. A temporary/mobile-friendly web flow is sufficient for the first implementation.

### Current implementation boundary

This section is architecture only.

Do not implement yet:
- screening persistence;
- candidate magic-link pages;
- recruiter screening UI;
- WhatsApp/email delivery;
- matching mathematics;
- React Native screening.

Next implementation work should first align the resume/intake contract with this architecture, then introduce normalization/screening in small verified blocks.

## Candidate profile presentation foundation — 2026-10-07

This section records the next product simplification after the screening architecture baseline.

### Start with the basic profile

Implementation should begin with the smallest predictable candidate profile rather than a large all-in-one CV model.

The first profile block should focus on data that the parser can extract reliably and that recruiters need immediately, for example:
- first name / middle name / last name;
- email / phone;
- location where useful;
- date of birth / nationality / gender when explicitly present and product/legal requirements allow;
- languages and stated proficiency;
- other simple stable personal facts that are directly supported by the source.

Do not make the first implementation depend on the final occupation/skill/matching model.

### Separate experience from occupations

Work history and normalized occupations are different concepts and should remain independent.

Experience is source history:
- employer when present;
- original/source title when present;
- start/end period;
- original description;
- translated display description where needed.

Occupation is a normalized professional interpretation:
- backed by ESCO;
- derived mainly from title + duties + work context;
- not inferred from employer name alone;
- may consolidate several differently named jobs into one professional occupation.

A candidate may have multiple experience records that normalize to the same occupation. One experience record may also support more than one capability area.

### Occupations and skills are separate product patterns

Do not mix occupations and skills into one visual or screening list.

Occupations answer:
- what professional roles has this person actually performed;
- for approximately what timeframe;
- which normalized ESCO occupations describe that experience.

Skills answer:
- what concrete capabilities can this person perform;
- which are directly evidenced by the CV/input;
- which were suggested by ESCO and later confirmed/rejected through screening.

This separation is a core readability and screening principle.

### Short profile vs detailed profile

The product should support two views of the same candidate.

Short profile:
- compact identity/contact/language facts;
- normalized occupations;
- occupation experience timeframe;
- confirmed/high-value skills;
- minimal status needed for recruiter decisions and future matching.

Detailed profile:
- full experience timeline;
- employer/source title/dates;
- original and translated descriptions;
- education/certifications;
- screening details;
- original resume/document as source evidence;
- other supporting facts.

Recruiters should be able to understand the candidate from the short profile without reading the full CV.

### Timeframe is a first-class capability signal

Experience duration should be derived from work-history intervals and associated normalized occupations.

The product should prioritize readable duration such as:
- `3y 8m`;
- `1y 2m`;
- `6m`.

Do not naively sum overlapping periods. Chronological overlap must be merged so simultaneous jobs do not double-count calendar experience.

Skill duration may be shown only when supported by evidence that connects the skill to one or more dated experience intervals.

If a skill is only confirmed through screening/ESCO suggestion and no historical interval is known, represent it as confirmed without inventing a duration.

### Seniority wording is contextual, not a simple year threshold

Do not define universal:
`X years = Junior/Middle/Senior`.

Years are an important signal but not sufficient proof of seniority.

Future level presentation may combine:
- duration;
- recency;
- responsibility evidence;
- role complexity;
- leadership/supervision evidence;
- occupation-specific conventions.

The visible label should fit the profession. Generic wording such as `Basic / Experienced / Advanced` may be more appropriate for many operational occupations, while occupation-specific labels such as `Junior / Mid / Senior` can be used where they are meaningful.

Exact seniority scoring remains a later design block.

### Visual direction

The primary recruiter view should emphasize:
1. occupations;
2. timeframe;
3. skills;
4. languages / key requirements.

A future compact timeline or visual pattern may show occupation duration across calendar time, with detailed employers/descriptions available only on expansion.

Do not optimize the first implementation around large data tables. The product goal is rapid recruiter comprehension.

### Implementation order

Proceed progressively:

```text
1. Basic Candidate Profile
        ↓
2. Experience source history
        ↓
3. Occupation normalization
        ↓
4. Skills as a separate capability block
        ↓
5. Screening confirmation
        ↓
6. Short profile + detailed profile presentation
        ↓
7. Matching / seniority / graph-distance mathematics
```

Keep each step independently understandable and verifiable before adding the next layer.

This section is architecture/product direction only. No frontend, persistence, matching or screening implementation is introduced by this documentation change.


## Candidate Processing audit checkpoint — 2026-10-08

Cross-project code/document audit captured in `docs/planning/CANDIDATE_PROCESSING_AUDIT_2026_10_08.md`.

This is a **documentation checkpoint**, not a change to AI-BENCHMARK-1 implementation scope or status. The local benchmark remains IN PROGRESS; no production Candidate domain, authorization, schema, file intake, ESCO mapping or screening implementation has been approved by the audit alone.

Before the next implementation block: review benchmark evidence; obtain owner approval for Candidate domain repository ownership, minimal Candidate core, artifact policy and consent/access boundary. Do not promote development ResumeDraft endpoints to production.

## CV-UPLOAD-PREVIEW-1 — parallel development-only vertical slice (2026-10-08)

Status: **VERIFY / owner local check pending**.

Owner approved a simple connection of Recruitment `Add Candidate` to existing parsing logic through one-file drag/drop. This does **not** approve production Candidate identity, a Candidate-Organization relation, or extended permissions. Keep AI-BENCHMARK-1 research status unchanged.

Implemented in this backend:
- `POST /api/dev/resume-drafts/preview-file` for binary PDF/DOCX (8 MiB max) using Express raw body and an encoded filename header;
- development-only routing; session-cookie authentication, unlocked session and Organization membership required; small request rate limit, restricted local browser origins;
- MIME/extension and basic signature checks; oversized/unexpected input fails early;
- OpenAI provider file parsing through existing `candidate.resume.parse@1` + strict `ResumeDraftV1` validation;
- returns an ephemeral JSON preview, with no ResumeDraft/Candidate/artifact database writes;
- tests for disabled route, session/organization boundary, invalid input, size, valid preview and no persistence.

Limitations: basic PDF/ZIP magic checks are *not* antivirus or full DOCX verification; this is **not** a production source-artifact ingestion path. No original file retention, malware scanning, screening, consent workflow, retry job or Candidate creation. This feature is only wired when AI_PROVIDER=openai and OPENAI_API_KEY is configured. Ollama remains text-only.

Do not deploy publicly or use real third-party CVs without the proper permission/privacy controls. Before production Candidate ingestion, follow Gate 0 and the Candidate Processing audit.

Verification:
- CI is a code-level check; the owner must still run `npm run verify`, then test in-browser through the Vite proxy with a permitted synthetic/sample PDF or DOCX.
- Keep in VERIFY until a local browser test succeeds.


## CANDIDATE-CORE-SCHEMA-1 — isolated foundation (2026-10-08)

Status: **VERIFY — owner local DB migration pending**. Owner approved strict separation of Candidate identity from contacts and future relationships/experience/audit. Central `HigaBase_Plans/CANDIDATE_ARCHITECTURE.md` §20 authorizes the structure; backend `PROJECT_RULES.md` grants a narrow schema-only exception.

Implemented: `Candidate` → `hb_candidates`, `CandidateContact` → `hb_candidate_contacts`, append-only migration `20261008192000_candidate_core_foundation`. No candidate API and no exposure through Recruitment yet. No organization relation or event log yet: therefore no audited Candidate creation should be permitted.

Acceptance: owner runs `npm run db:generate`, `npm run verify`, applies controlled migration (`npm run db:deploy` or approved Prisma equivalent), and runs `npm run db:status`. The CI may test schema/build but does not confirm the owner's DB. Later block must add actor/organization-scoped transactional audit, consent request and access guards before opening create/list/detail API. Preserve AI-BENCHMARK-1 existing state.

## DEV-CANDIDATE-PIPELINE-A1 — experimental link foundation (2026-10-09)

Status: **IN PROGRESS → VERIFY after source commit; owner local migration/build pending**.

Owner approved an explicitly disposable local-development Candidate Pipeline to iterate over 5–15 CV documents. The owner reported `npm run db:status` green with 15 migrations on local PostgreSQL `higa_systems`. This authorizes a *narrow first block*: prepare an isolated relational experiment link between the approved Candidate Core and existing ResumeDraft research entity, without activating Candidate creation, list, detail, reset endpoints or changing the production contract.

Technical design for A1: append `CandidateDevIntake` as a strictly research-only link containing a UUID ID, required candidateId, required resumeDraftId, creation timestamp, uniqueness per resumeDraft, FK cascade upon Candidate/Draft removal. `Candidate` and `ResumeDraft` receive only inverse relations, no new business fields. Physical table `hb_candidate_dev_intakes`. This is not an Organization relationship, authorization policy, screening/consent state, Resume source model or production audit. It allows future tracked cleanup of specifically experimental rows rather than truncating business tables.

Safety: schema-only; no write API, no enabled reset, no files written, no Candidate/Profile data changes, no cross-Organization reads. Test locally via Prisma generate, verify, db deploy and db status *only after* reviewing migration. No real candidate data upload/creation authorized by A1. Next A2 requires an explicit limited-development API/actor-access and transaction design, with tests, before any writes.

## DEV-CANDIDATE-PIPELINE-A1 — source landed / VERIFY (2026-10-09)

Express `main` commits: `6addfeff0a8f1a7c6247e65a18a77d0d541e0ea5` (schema), `bb1f6111057f2727567ff08a95dfd0719dbf907e` (migration). Added `CandidateDevIntake` / `hb_candidate_dev_intakes` with unique ResumeDraft FK and Candidate FK; Candidate Core unchanged apart from inverse relation. The owner-reported local status **before this change** was 15 migrations applied and database up-to-date. After this commit there are 16 migration folders in source; the new migration is **not yet confirmed applied locally**.

Status: **VERIFY**. Source changes only; local `npm run db:generate`, `npm run verify`, `npm run db:deploy`, `npm run db:status` not executed by agent. Existing 15 migrations, ESCO, user/organization records, React and business authorization unaffected. This does not add an intake API, link rows, candidates, drafts, reset endpoint or Candidate-Organization relation. The next implementation block A2 must be separately scoped after owner verifies A1.

Owner in `higa_systems_express`: `git pull` → `npm run db:generate` → `npm run verify` → `npm run db:deploy` → `npm run db:status`. Stop and report errors; do not reset or drop the database.

## DEV-CANDIDATE-PIPELINE-A2 — experimental write (2026-10-09)

Status: **VERIFY — owner local build/tests and manual API validation pending**.

Owner explicitly agreed to proceed after A1. A1 owner evidence: 16/16 migrations applied, database up to date, 34 test files/143 tests, TypeScript and build green. This evidence applies to A1 only.

A2 committed on Express `main`:
- `src/resume-draft/candidate-dev-intake-repository.ts`: one Prisma transaction creates Candidate, optional CandidateContact fields, full ResumeDraftV1 JSONB and CandidateDevIntake link;
- `src/resume-draft/routes.ts`: adds `POST /api/dev/candidates/intake` to the same PDF/DOCX/authentication/session/organization-workspace/Origin/rate-limit checks as the preview route; requires first and last name and returns candidateId/resumeDraftId only;
- `src/dependencies.ts`: wires the repository only with OpenAI parsing outside production;
- `tests/resume-draft/file-preview.test.ts`: adds dev intake authenticated happy-path test (further negative cases and DB integration still required).
- Preview endpoint remains non-persistent. No React changes, no new migration beyond A1, no ESCO/Permissions/Organization relationship changes.

This is research-only ingestion; real third-party CV use requires appropriate consent/legal basis. It does not persist original PDF/DOCX, does not create verified/screened state, does not deduplicate identity and must never be promoted to a production API as-is.

Verification requested: owner `git pull` in Express and `npm run verify`. Do not claim verified unless actual local output confirms; no automated/local verification was run by assistant. Follow with explicit synthetic/sample CV API testing and inspect persisted data before Block B.

## DEV-CANDIDATE-PIPELINE-B — experimental listing API (2026-10-09)

Status: **VERIFY — owner local test and browser confirmation pending**.

On Express `main`, `CandidateDevIntakeRepository.list()` reads up to 100 most recent explicitly linked experimental Candidates, including contact. `GET /api/dev/candidates` exposes those rows with session authentication, unlocked workspace, organization membership and local Origin checks; disabled outside development. No production scoped Candidate list, no organization relation, no permissions implementation, no new schema or migration, no source file preservation. Country remains empty because the minimal Candidate Core has no country. Route tests added for authenticated listing, unauthenticated and unavailable cases.

Security boundary: this local development-only list is global across experimental records; do **not** deploy, expose externally, or repurpose as a production read API. It has no organization-specific consent/access scoping.

Owner verification: `git pull`, `npm run verify`, `npm run db:status`, then restart Express. No local test execution was performed by the assistant. A successful owner browser row check is still required.

## DEV-CANDIDATE-PIPELINE-C — experimental detail API (2026-10-09)

Status: **VERIFY — owner local checks pending**.

On Express existing `main` only: `CandidateDevIntakeRepository.findDetail(id)` reads one experimental Candidate via the CandidateDevIntake link, with CandidateContact and ResumeDraftV1 validated from JSONB. `GET /api/dev/candidates/:id` requires dev mode, authenticated/unlocked workspace, at least one organization membership, permitted Origin and valid UUID; it returns 404 if not found. No organization consent scope exists; **never enable as production API**. No database migration or data mutation in this block. Read/missing/auth tests added.

Owner: `git pull`, `npm run verify`, `npm run db:status`, restart backend. This source-level implementation is NOT locally verified by agent.

## DEV-CANDIDATE-PIPELINE-D — guarded experimental reset (2026-10-09)

Status: **VERIFY — local owner typecheck, tests and reset behavior pending**.

Owner approved a development-only reset/repeat workflow, without React changes. Express `main` commit `70f5ee185e807269dc13f49a19657385f9f939c5` introduces:
- `npm run dev:candidates:reset` → `tsx src/candidate-dev/reset-cli.ts`;
- `reset-safety.ts` requiring non-production execution and `postgres://` / `postgresql://` URL on localhost/127.0.0.1/::1 to database `higa_systems`;
- interactive TTY-only confirmation `RESET N` after displaying test intake count; no silent automated reset;
- a Prisma transaction selecting exclusively `hb_candidate_dev_intakes` rows, deleting their Candidate records (with FK-cascaded contacts and links) and their associated ResumeDraft rows; unexpected counts abort and roll back;
- unit tests for allowed/disallowed targets.

No schema/migration modifications; no system users, organizations, locations, ESCO, other unlinked Candidate or unlinked research ResumeDraft records intentionally targeted. This is destructive **only to linked experimental records** and cannot be undone after commit. Do not run it before reviewing current experimental Candidate records. Strong local guard is not a replacement for proper production RBAC/consent/security.

Owner checks in `higa_systems_express`: `git pull`, `npm run verify`, `npm run db:status`. Run `npm run dev:candidates:reset` only when deliberately ready to erase all linked test Candidates; otherwise cancel at confirmation. Confirm with browser reload that table empties if reset was approved. No local tests or DB reset were run by agent; keep VERIFY until owner confirmation.

## End-of-day reconciliation — 2026-10-09

[Cross-project evidence and next steps](../../docs/operations/CANDIDATE_DEV_CHECKPOINT_2026_10_09.md). Owner verified Express main for development Candidate Blocks A–C and Reset guard: `npm run verify` 35 files/150 tests, Prisma validate/tsc/build green, 16 applied migrations and current PostgreSQL schema. Browser/pgAdmin showed successful Candidate, CandidateContact, ResumeDraft and linked records; Recruitment list/detail worked. The previous *VERIFY pending* wording in the earlier A/B/C paragraphs is historical and superseded by this owner evidence; **development vertical slice verified, not production ready**. D reset guard confirmed listing four linked candidates and cancellation without deletion; actual destructive reset remains **VERIFY**. Rate limit 5 requests/15min observed; proposed 30/15min NOT applied. No Express code modified in this documentation checkpoint.

## DEV-CV-HARDENING-1 — development upload throttle (2026-10-10)

Status: **VERIFY — owner local check pending**. Existing Express `main` commits `3561f35234a4bb33f153e6e4cf10e7f4ea0fc7cd` and `bff33e225c9440007d30c459e269343f348b6aa5` increase **only the shared development CV Preview/Save limiter** from 5 to **30 per 15 minutes**; test in `tests/resume-draft/file-preview.test.ts` asserts shared budget across routes and HTTP 429 on the 31st request. No auth/global limit, AI provider, Prisma, migration, ESCO, production API or database contents changed. Owner must `git pull`, `npm run verify`, restart Express and check browser 429 behavior. Assistant did not run local tests.

## CANDIDATE-ESCO-RESEARCH-1 — read-only Candidate ESCO preview (2026-10-10)

Status **VERIFY — owner local and browser checks pending**. Owner approved only visual experimentation with three blocks: existing AI Extraction, ESCO Occupations, ESCO Skills. Express main adds `src/candidate-dev/esco-research.ts`, read-only search on the current `ResumeDraftV1.workHistory[].title` and `skills[]` using existing `EscoKnowledge.search`, verified whole-term equality in English, cap 6 occupations / 40 source skills, related ESCO skills sourced through existing `EscoOccupationSkill` with ESSENTIAL/OPTIONAL and labels/URIs from real concepts. `GET /api/dev/candidates/:id` attaches `candidate.esco` after its existing dev-only session/workspace checks. Test asserts enrichment called and returned. No new AI calls, similarity benchmark, migrations, Candidate write, verification or production contract. Conservative exact-term search may produce empty ESCO proposals for many CVs; inspect real results before proposing a semantic mapper. Owner must run Express `npm run verify` and real browser verification. No assistant-side execution on owner machine.

## CANDIDATE-ESCO-TIMELINE-1 — work-history research projection (2026-10-10)

Status **VERIFY — owner's npm run verify/browser check pending**. Replaced global experimental ESCO list with read-only `candidate.esco.experiences[]` aligned by temporary `ResumeDraftV1.workHistory` `sourceIndex`. Each original work period owns `occupations[]`, `directSkills[]` (CV-wide AI skill only if its literal term appears with word boundaries in that period's title/description), and `relatedSkills[]` (ESCO graph associations for that period's proposed occupations). Unattributable CV-wide skills remain `unassignedSkills[]`; none are candidate-confirmed. Missing dates remain missing; source CV, ESCO schema/graph, Prisma models and AI parser unchanged. Conservative EN whole-term search cannot semantically derive Recruiter from compound titles or identify competencies embedded in prose; display empty proposals rather than invent links. Added research unit tests + adjusted route mock contract. No migrations, writes, new AI calls or benchmarks. Run local Express verify and inspect browser outcomes before status DONE.

## ESCO Timeline TS strict access hotfix — 2026-10-10

Status **VERIFY (owner rerun required)**. Fixed 16 owner-reported TS strict/noUncheckedIndexedAccess errors (4 in `src/candidate-dev/esco-research.ts`, 12 in `tests/candidate-dev/esco-research.test.ts`): runtime guards for indexed work-period records; explicit non-null assertions only for known fixture entries in tests. No ESCO model or graph changes, migrations, new AI logic, or frontend change. Owner to `git pull` Express main and `npm run verify`; no claim of green until owner confirms.


## 2026-10-10 CV intake and AI–ESCO research

AI-to-ESCO integration discovery (2026-10-10): Existing `AiRuntime` and OpenAI Responses Provider handle only `candidate.resume.parse@1`; current candidate ESCO research uses synchronous lexical ESCO search and graph relations, with no Uno/AI classification task. `AiRuntime` exposes provider/model/input-output tokens/duration, but `ResumeDraftService.previewFile` returns only draft; dev intake persists only draft and Candidate. Historic per-CV billable cost therefore cannot be shown accurately. Benchmark pricing is model-specific ESTIMATE, not an invoice. Proposed next implementation: bounded per-WorkExperience AI selection from REAL ESCO candidate IDs via a new typed provider task, experiment read-only, rate limits and cost control; separate operation usage persistence needed for defensible per-candidate cost. No AI integration coded in this step.


## AI ↔ ESCO Research V1 (2026-10-10)

2026-10-10 AI↔ESCO Research V1 — STATUS VERIFY (owner local tests pending). Added provider task candidate.esco.select@1 and candidate.esco.skills.select@1 via OpenAI Responses, using existing AiRuntime. Experimental CandidateEscoAiResearchService obtains bounded candidate occupations via real EscoKnowledge.search EN prefix (<=12 query roots per work period; <=70 candidates), invokes AI for <=2 occupations and server-whitelists IDs; uses existing EscoOccupationSkill associations then AI selects up to 12 relevant real skill IDs per period, also server-whitelisted. Up to 8 work periods, research triggered ONLY by authenticated, unlocked, workspace-authorized, dev-only POST /api/dev/candidates/:id/esco-classify (rate-limit 3/15min), never by Candidate GET. No canonical ESCO mutations, migrations, Candidate writes, extra CV upload, or AI cost storage. ESCO skill origins AI_PROPOSED vs lexical DIRECT. Note possible sparsity due lexical candidate recall; no guaranteed classification. Added unit tests for ESCO ID whitelist, isolation and missing dataset. Run npm run verify locally; do not claim green. API currently synchronous paid calls; UI waits on explicit button. Other providers unavailable for this experimental task.


## ESCO concept tooltip and cost research

2026-10-10 ESCO Popovers + iCosts checkpoint — **VERIFY, owner npm run verify/browser pending**. Experimental Candidate Details now uses shared `EscoConceptBadge` with React-Bootstrap Popover on hover/focus for all ESCO occupation/skill/related/unassigned badges. Fetches official description lazily through new read-only `GET /api/esco/concepts/:id/description` using active dataset's EN/NL DESCRIPTION/DEFINITION/SCOPE_NOTE, cached per concept in React. Displays clear fallback if description absent; never uses whole CV text as description. Existing graph and migrations unchanged; added endpoint route test. Cost: no feature change; current historic Candidate says `Not recorded`; accurate per-candidate iCosts require collecting operation model + actual input/output tokens, pricing tariff/version and operation ID at execution; Billing can reconcile totals, cannot reliably allocate historical charges to candidate. No API/admin key requested. Visual and real description content needs owner check.


## Occupation Discovery V2 research (2026-10-10)

2026-10-10 Occupation Discovery V2 + skill discovery experiment — **VERIFY, local tests and real OpenAI run pending**. In Express main, added OpenAI task `candidate.esco.discover@1`: for each existing Work Experience the model proposes bounded English occupation lookup phrases and concrete skill lookup phrases from title/duties, no fabricated ESCO IDs. Service queries existing `EscoKnowledge.search` for Preferred/Alternative labels (prefix search) using complete phrases rather than blindly truncated 5-letter tokens, builds max-70 real occupation candidates, retains existing `candidate.esco.select@1` strict allowlist, then searches independent SKILL concepts as well as related skills and uses `candidate.esco.skills.select@1` to propose up to 12 real skill IDs. An experience may thus have skill proposals even if occupation discovery found none; everything is per sourceIndex and ephemeral. Candidate UI unchanged, existing proposed badge and official descriptions. No DB migrations, writes, ESCO graph/schema changes, human confirmed state, iCosts changes, or autosave. English prefix may still miss synonyms that do not begin with a proposed query, and results may vary per model run. Explicit button still causes paid and synchronous work; rate limit 3 per 15 min. Unit tests updated and new missing-occupation/skill scenario added; **owner must git pull Express main and npm run verify** and then inspect results.


## 2026-10-10 Occupation Discovery V2 fixture isolation hotfix

**VERIFY — owner rerun pending.** Owner reported 1/157 failing test (`tests/candidate-dev/esco-ai-research.test.ts`): mock `candidate.esco.discover` always returned `recruiter` queries even for `Warehouse worker`, so mocked ESCO search introduced candidate 42 in wrong period. Fixed only the test fixture to respond to work-period title; strengthened assertion that second discovery receives `Warehouse worker`. No production ESCO resolver, AI algorithms, database, or React change. Express main commit `517759ec174390452f6f3af8f4582943734154c5`. Owner to `git pull` and `npm run verify`. Do not claim green until confirmed.


## Context Verification V1 — 2026-10-10

2026-10-10 CONTEXT-VERIFICATION-V1 — **VERIFY, owner local tests pending**. Experimental Express AI/ESCO occupation selection now loads EN DESCRIPTION (fallback DEFINITION) from active ESCO concept versions for up to 70 discovered occupation candidates; passes preferred label, actual matched Preferred/Alternative term, official description (<=1100 chars) and actual Work Experience title/duties to existing typed candidate.esco.select task. OpenAI instructions prioritize documented duties, avoid unsupported manager/operator/supervisor seniority, and reject unsuitable occupations. Server validates selected ID against verified active-dataset occupation concepts; no new tables, migrations, changes to canonical ESCO data, persisted classification, cost accounting or React page. Existing per-period boundaries maintained. Test fixture now covers description+matchedTerm provided to AI selector. Important: no live AI accuracy claim until owner runs Express npm run verify and compares saved CV. Caveat: upstream prefix recall and missing official descriptions remain possible limitations.


## Context Verification V1 test fixture hotfix — 2026-10-10

**VERIFY; owner rerun pending.** Owner reported 156/157 passing, failure in `tests/candidate-dev/esco-ai-research.test.ts` after official ESCO descriptions were added. The mock `escoConceptVersion.findMany` returned description-shaped rows for both occupation metadata and related skill labels. Updated the fixture to distinguish `include.texts` from `include.labels`; hardened related label access with optional chaining in `src/candidate-dev/esco-ai-research.ts`. No ESCO data/schema migration, frontend change or classification logic change. Express main commits `4f204943f45f5bfcce1c3f448eb922f0d7141d0a`, `6f4b291d209e7739b6bb4d9e6720e186fa960595`. Owner should `git pull` and run `npm run verify` before claiming green.


## Occupation Discovery instruction experiment and diagnostics — 2026-10-10

Status **VERIFY — owner local npm run verify and live browser run pending**. User showed CV: Warehouse Employee → Warehouse worker, Electromontage → Electrician; Manager-consultant with retail sales/customer advice/payment tasks → no occupation. Changed only OpenAI discovery prompt to prioritize documented duties, realistic Preferred/Alternative occupation search phrases and avoid unsupported managerial seniority; no canonical ESCO mutations. Experimental read-only AI response adds per-work-history `diagnostics` with bounded occupationQueries, retrievedOccupations (real concept ID, label, matchedTerm) and selectedOccupationIds; React CandidateDetails displays them only inside collapsed Research Diagnostics for that same period. Added regression assertions for fixture timeline isolation. Nothing is persisted; results remain ephemeral and paid calls remain explicit. This is observability, NOT guaranteed improvement; owner to pull BOTH Express and React main, run both `npm run verify`, restart services and test one CV. Do not infer exact reason for a failed match without diagnostics.


## Esco Intelligence Bridge V1 — 2026-10-10

**VERIFY; owner local checks pending.** Added `src/esco/bridge/work-context.ts` as internal typed, read-only intermediary between Uno AI and existing ESCO graph. Existing `candidate.esco.discover@1` now emits **structured interpretation of one work period** (`coreActivities`, `responsibilities`, `equipment`, `workingConditions`, `evidenceGaps`) together with bounded occupation/skill ESCO lookup phrases, avoiding another model call. Work context is stored **only in ephemeral research API response** (`candidate.esco.experiences[].interpretation`) and displayed in collapsed `Uno Interpretation` for each Work Experience separately, alongside existing Research Diagnostics/ESCO suggestions. No DB/schema/migrations, ESCO alterations, auto confirmations, persistent AI decisions, billing changes or changes to candidate upload. Updated test mocks for typed discovery output and assertion on context; all test/build results **unverified pending owner `git pull` and `npm run verify` in BOTH Express and React**. Isolation still limits processing to first 8 work periods as previously. This is an observability/interpretation foundation, not proof of improved matching or semantic search. Compare interpretation, retrieved candidates and final ESCO selections on same CV to isolate failure source.


## Esco Bridge strict optional type hotfix — 2026-10-10

**VERIFY — owner rerun needed.** Owner's Express `npm run verify` failed at TypeScript `TS2379` in `src/candidate-dev/esco-ai-research.ts:65` because `ResumeDraftV1.workHistory[].description` may explicitly be `undefined` under `exactOptionalPropertyTypes`, whereas helper `workContextInput` accepted optional `string | null` only. Updated helper input annotation in `src/esco/bridge/work-context.ts` to `description?: string | null | undefined`. Runtime still normalizes absent descriptions to empty text; AI/ESCO logic, database and frontend unchanged. Express main commit `4dcf4e51fa070ae0cbc87fd0c251f96b673348a0`. Owner to `git pull` and `npm run verify`; do not claim successful verify until result received.


## 2026-10-10 ESCO Bridge AI_OUTPUT_INVALID oversized context hotfix

**VERIFY — owner local checks pending.** Live dev POST `/api/dev/candidates/:id/esco-classify` failed at `candidate.esco.discover@1` with `AiExecutionError: context.responsibilities/equipment/evidenceGaps: too_big`. The provider JSON schema constrained object shape but not array maximums; strict Zod `workContextSchema` rejects arrays exceeding 6/6/4 respectively. In Express `src/ai/openai-responses-provider.ts` strengthened instruction with numeric limits and added narrowly scoped `boundBridgeDiscoveryOutput` for discovery output only: truncate oversized arrays (and overlong string items) to existing contractual bounds prior to `AiRuntime` validation. This does NOT silence missing fields, wrong field types or invented ESCO IDs; strict validation remains. Added provider regression test with 9 responsibilities, 8 equipment, 7 gaps. No schema/migrations/ESCO graph, React, other AI tasks or iCosts changes. Main commits `1b14351315b644a8d427e81c7d6fea79e3ae726f`, `6a70a54806932919f8313a3fb61244609665ec7f`. Owner to `git pull`, `npm run verify`, restart Express and rerun explicit research once; repeated AI actions may incur cost. Do not call verified until owner reports green.


## ESCO Bridge V1 — Evidence Mapping checkpoint (2026-10-10)

**VERIFY, owner local checks pending.** Created `src/esco/bridge/evidence-mapping.ts`: deterministic conservative mapping of already extracted `ResumeDraftV1.skills` and `.tools` to one `workHistory[sourceIndex]` ONLY if a normalized whole-word/whole-phrase literal occurs in that job's title/description. No inference from other periods, no fuzzy semantic claims, no new AI calls, and no confirmations. Attached `experience.evidence` to temporary AI ESCO response, added collapsed `Uno Evidence Mapping` to CandidateDetails. Up to two source-linked skills feed existing bounded SKILL label retrieval alongside max-six AI skill queries; CV-wide tools are displayed as evidence but not automatically treated as ESCO skills. New tests check ETL/Docker vs Forklift period isolation and non-match of SQL inside NoSQL. No migrations, ESCO graph/source changes, persisted candidate skill decisions, or iCosts changes. Limitations: paraphrases and abbreviations without literal matching are deliberately not assigned; evidence strings currently reflect matched CV-wide terms, not character offsets. Owner should pull BOTH Express and React main, run `npm run verify` for each, and browser-check per-period Uno Evidence Mapping and skill proposals. Do not claim tests passed until owner confirms.


## ESCO Intelligence Bridge V2 — contextual candidate ordering (2026-10-10)

**VERIFY; owner local tests pending.** First stage of Bridge V2 implemented in Express main without migrations or changing ESCO datasets. New `src/esco/bridge/occupation-ranking.ts` ranks ONLY already-retrieved, active-dataset verified ESCO occupation candidates via conservative normalized token overlap of each period's title, duties, structured Uno context against candidate preferred/matched labels and existing EN official description. Deterministic ordering with concept ID tie-break; results still passed through existing Uno occupation selector and server-side ID allowlist. The per-period `Research Diagnostics` retrieved occupation order reflects this ranking; no artificial confidence percentages, no persistent classification, no new ESCO concepts, and no unrelated work-history context. Test added for warehouse workforce coordinator vs court jury coordinator, plus empty candidate pool. Important limitation: this is **reordering, not semantic retrieval**: cannot find occupations missing from lexical search, has simple lexical scoring only, and must not be described as measured quality improvement. No changes to React or AI provider/extra model calls. Owner to git pull Express main, npm run verify, restart Express and browser test on unchanged CV with per-period diagnostics; if recall remains poor, next step consider bounded alternative candidate generation rather than DB migration.


## ESCO Bridge V2 — bounded lexical fallback retrieval (2026-10-10)

**VERIFY — owner local checks pending.** Approved continuation on existing `main` only. Express adds `src/esco/bridge/occupation-retrieval.ts` and uses it only when the original occupation pool contains fewer than 10 concepts. Up to six deduplicated single-word queries are extracted deterministically from the CURRENT work-period title, at most three Uno core activities and at most two responsibilities; generic role words are excluded. Each query uses the existing `EscoKnowledge.search` with max 10 results. The pool remains capped at 70 unique server-provided ESCO concepts; existing active-dataset/kind validation, contextual ordering, Uno selection and ID allowlist remain in place. `Research Diagnostics.occupationQueries` now includes fallback queries. Added isolated tests for title fallback, current-period terms, limits and deduplication.

**Limitations:** This is a conservative **lexical fallback**, not substring/description search or full semantic retrieval; a fallback query may retrieve noisy concepts and does not prove better recall. First 70 candidates may still suppress later matches; the old cap/order problem remains unresolved. No canonical ESCO mutation, DB/Prisma migration, React change, new model invocation, result persistence or confirmation. Tests and behavior **not executed/verified locally by agent**. Owner: pull Express `main`, run `npm run verify`, restart Express, explicitly rerun research on the same CV (paid AI call), compare Street Paver / Logistics Operator — DAF / Coordinator candidate counts and final selections; report regressions. Do not mark DONE before verification and empirical comparison.


## ESCO Hybrid Retrieval V1 — official description discovery (2026-10-10)

**VERIFY — owner PostgreSQL/runtime tests pending.** Owner approved one bounded Express-only block on existing `main` (no branches or PRs). Added `PrismaEscoKnowledge.searchOccupationDescriptions` in `src/esco/knowledge.ts`: up to 12 English keywords from a single work-period context form a bounded OR full-text query over the ACTIVE dataset's official EN `DESCRIPTION` entries. Query excludes ISCO taxonomy via `OCCUPATION` + `hb_source_family='occupation'` and returns real canonical concept IDs/URIs with official preferred English labels; no fabricated IDs. No new index or tables; the on-the-fly tsvector must be benchmarked before production. Existing `EscoKnowledge.search` prefix behavior remains unchanged for all other consumers.

`src/candidate-dev/esco-ai-research.ts` now merges at most 25 full-text results with the lexical pool before the current contextual ranker and Uno allowlisted selector. It first reserves 45 places for lexical discoveries and then admits description hits into the fixed 70-concept total, filling any remaining capacity with original lexical results. This is a research compromise, **not** an optimized multi-channel scoring policy, and its order/recall must be measured; a description hit may be absent due to SQL cap, missing DESCRIPTION, or word mismatch. Existing period isolation, user-triggered AI calls, no persistence, no candidate confirmation, and independent Skills path remain. No React change, no new migrations, no canonical ESCO alteration, and no embeddings. Added `tests/esco-description-retrieval.test.ts` covering mocked concept identity and empty text.

**Verification not yet run on the owner's environment.** `npm run verify` and a real PostgreSQL check are required; specifically measure full-text query latency under representative CV lengths and inspect whether exact good occupations survive (poultry breeder, fruit and vegetable picker) and missing ones improve (Coordinator, Street Paver, Logistics Team Leader). Old labels/description origin is not yet exposed distinctly in Research Diagnostics, so do not claim that provenance is fully instrumented. Any slowdown or excessive noise blocks closure. Do not mark DONE until owner green tests, database timing and browser evidence. React/Prisma/production Candidate model unchanged.


## ESCO Bridge V3 — diagnostic occupation evidence (2026-10-10)

**VERIFY — local owner test and real-CV assessment pending.** Owner approved diagnostic-only work on existing Express `main`, no branches/PRs. Added `src/esco/bridge/occupation-evidence.ts`: for the first 12 ranked, active-dataset ESCO Occupations per work period, compare normalized terms from the work-period title/description and Uno context to the official EN description and preferred label. Reports bounded `supportedTerms`, `unmatchedContextTerms`, `descriptionAvailable` and one of `TEXT_OVERLAP`, `NO_TEXT_OVERLAP`, `NO_DESCRIPTION`. Attached optional `diagnostics.evidenceDiagnostics` to the ephemeral research API contract; tests added in `tests/occupation-evidence.test.ts`. No frontend surface has been added yet: the diagnostics are only in the response payload.

**Crucial limitation:** This is **lexical evidence inspection only**, NOT actual semantic entailment, contradiction detection, proof of a profession, AI confidence, a quality gate, or a replacement for the current ranking. Absence of matching terms does NOT mean contradiction. No changes to occupation ordering, Uno selection, skill selection, whitelist, ESCO dataset, PostgreSQL schema/migrations, React or confirmed Candidate data. No new AI call. Verification has not been run by the agent or owner for this block. Owner: pull Express main, run `npm run verify`, test the existing dev research API on the same saved CV and inspect `diagnostics.evidenceDiagnostics` against Coordinator, Logistics Team Leader, Warehouse Employee and Electrician; compare before any approval to use this in selection. Mark DONE only after evidence.


## ESCO Evidence diagnostic visibility — selected concept inclusion (2026-10-10)

**VERIFY.** At owner's approval, Express `main` now appends an evidence diagnostic for each Uno-selected occupation that was not in the first 12 ranked diagnoses. It reuses existing already-loaded official description/context, adds no model request, does not modify selection or confirmed Candidate facts. Existing `evidenceDiagnostics` remains optional in the response. Commit `ae20ffdf`. Requires local `npm run verify` and a paid dev research run to verify selected IDs are always represented. UI display is documented in central React CURRENT_TASK. No DB or migrations.


## ESCO semantic retrieval experiment V1 — local Ollama benchmark (2026-10-10)

**VERIFY — source committed, local compiler/runtime and empirical accuracy not verified by agent.** Owner approved one isolated semantic experiment after Hybrid V1 and read-only lexical evidence diagnostics. Implemented in existing Express `main`, without a branch or PR: `src/esco/bridge/semantic-ranking.ts` deterministic cosine ranking and `src/esco/bridge/semantic-cli.ts` standalone explicitly-run read-only benchmark; `package.json` script `esco:benchmark:semantic`; `tests/esco-semantic-ranking.test.ts` for vector validation and ordering. There is **no** change to Candidate research service, Uno discovery/selection, existing ESCO searches, Skills Graph, API, React, DB schema or migrations, and **no persistence** of embeddings. No new npm dependencies.

Execution: use local development DB containing ACTIVE official ESCO **1.2.0** and local Ollama with an **embedding-capable model** already installed, e.g. `nomic-embed-text` (the existing gpt-oss / qwen generation models cannot be assumed embedding-capable). Run `npm run verify`, then `npm run esco:benchmark:semantic -- --model nomic-embed-text` from Express with DATABASE_URL and optionally OLLAMA_BASE_URL configured. CLI embeds only official English ESCO occupation preferred labels and DESCRIPTION/DEFINITION texts from the active source-family `occupation` dataset and six **synthetic, non-personal** work-period fixtures (workforce/staffing coordinator, poultry packer, intralogistics DAF, recruitment consultant, pharmacy assistant, cheese production). Computes cosine similarities and top-10 ESCO concept IDs for each fixture; reports time to construct the ephemeral embedding index. No real candidate CV, names, contact data or organization records are sent to Ollama by this CLI. No outbound SaaS API is built into this experiment.

IMPORTANT: benchmark `expectedIds` are **prior observed baseline selections**, NOT ground truth: coordinator (1933) and leather packer (3202) are known wrong prior selections. For DAF old baseline abstained; other baseline IDs were positively reviewed. Cosine scores are NOT match confidence. The prototype tests corpus-wide semantic **retrieval**, not semantic entailment or changes to Uno output. It has no caching, so it regenerates embeddings for all occupations on each explicit CLI run; measure time/CPU and do not use as production runtime. ESCO occupations without EN description/definition are omitted and not assessed; multilingual retrieval untested. Before any integration: owner must supply console output for six fixtures; manually evaluate profession relevance and baseline regression; test on larger held-out sample. Do not claim accuracy improved or mark DONE without evidence.


## ESCO Evidence Rerank V1 — frozen semantic candidates (2026-10-10)

**VERIFY — local owner run required; do not mark as accurate or DONE.** Owner approved a six-case evidence-based reranking experiment after semantic cosine top-10 returned domain confusions. Express `main` only; no branches or PR. Added `src/esco/bridge/evidence-rerank.ts`, `src/esco/bridge/evidence-rerank-cli.ts`, `tests/esco-evidence-rerank.test.ts`, package script `esco:benchmark:evidence`. Explicit CLI compares six synthetic non-personal work-period examples against **the frozen exact top-10 IDs observed in owner's preceding Semantic V1 report**: staffing coordinator, poultry packing, DAF intralogistics, recruitment, pharmacy, dairy. It retrieves official English preferred labels plus DESCRIPTION/DEFINITION for each allowlisted ESCO 1.2.0 occupation and makes one local Ollama generation request per fixture. AI must select at most two IDs from the strict supplied list or return `insufficientEvidence=true` with no selection. Rejects invalid / foreign IDs. Prints original semantic first ID, prior known wrong Hybrid selection (where available), optional reviewed positive reference, selected IDs/labels, model explanation, per-call runtime or error. This benchmark **does not search corpus**, integrate into existing selection, mutate ESCO/Candidate DB, write persisted vectors, or change React/Skills/production API.

Owner instructions: update Express `main` via `git pull`; run `npm run verify`, then run `npm run esco:benchmark:evidence` with local Ollama `qwen3.5:9b` running and development DATABASE_URL populated. Set `$env:ESCO_EVIDENCE_MODEL = 'qwen3.5:9b'` if desired. Six test cases may take time. Note: qwen3.5 generation model supports structured output only if Ollama accepts schema; failures must be reported without claiming completion. The earlier three good baseline IDs (946,3231,2539) are reference positives, not full independent ground truth. An empty selection might be desirable when retrieval missed the right role (especially DAF), not automatically a regression. Similarity numbers are not probabilities. Test reliability, false positives and absent good candidates against lexical baseline before proposing any integration. No agent access to owner local execution, so verification remains pending.


## ESCO Hierarchy-Aware Retrieval — Block A (2026-10-10)

**VERIFY — owner local tests and PostgreSQL behavior pending.** Explicit owner approval for Block A only: isolated read-only ESCO occupation retrieval on existing Express `main`, no migrations/schema modifications, no React, no CV/Uno pipeline switch, default maximum parent depth 1. Code paths: `src/esco/bridge/leaf-occupation-retrieval.ts`, `src/config/env.ts`, `src/dependencies.ts`, and `tests/esco-leaf-occupation-retrieval.test.ts`.

The separate `EscoLeafOccupationRetrieval.search(query)` uses existing `EscoKnowledge.search` (active dataset, English preferred/alternative label prefix hits, maximum 25), queries existing `EscoHierarchyClosure` for `OCCUPATION_HIERARCHY` and filters candidate occupations to zero-descendant (leaf) matches. Only when no leaf label hit exists, candidate parents with an actual hierarchy descendant are eligible as lexical fallback, bounded by `ESCO_OCCUPATION_MAX_PARENT_DEPTH` (0..2, default **1**). This is **not** proof of CV occupation match: no semantic evidence, AI selection, matching percentages, skills confirmation, persistence, or new endpoint. The current CV AI research continues unchanged. Parent fallback is from the already retrieved lexical candidates; this block does not implement global leaf retrieval, related-occupation expansion, semantic ranking or user correction UI.

**Verification required:** run `npm run verify` on Express main, then measure retrieval on ACTIVE ESCO 1.2.0 using pharmacy assistant, warehouse worker/order picker, diplomat/ambassador and negative examples. Confirm English-prefix search limitations and no fallback past configured depth. No local `npm run verify`, DB or owner browser test claimed; status cannot advance beyond VERIFY. Central canonical architecture retains frozen ESCO graph and reference separation.


### Block A correction — leaf + bounded ancestor suggestion (2026-10-10)

**VERIFY — local validation pending.** Owner reported previous Block A `npm run verify` green (181 tests; 46 test files reported) and ran the isolated read-only PostgreSQL check: pharmacy assistant 3231, warehouse order picker 3372, ambassador 1172, embassy counsellor 2846 and forklift operator 3708 returned as leaves. Searches for `warehouse worker` and `diplomat` incorrectly returned only child matches (from alternative labels); `recruiter` was empty. The previous fallback condition did not provide the expected parent suggestions.

With explicit owner approval, Block A now selects the first ranked **terminal** concept from existing ESCO label retrieval, then independently fetches ancestor occupation concept(s) from `EscoHierarchyClosure` using `hb_descendant_concept_id=selected leaf` and actual `hb_depth>0 AND <= ESCO_OCCUPATION_MAX_PARENT_DEPTH` (default **1**). Parent preferred labels and official URI are resolved from the ACTIVE dataset's `EscoConceptVersion`. Primary result is tagged `parentDepth=0`; parent suggestions have their true graph depth. `DEPTH=0` yields leaf only; no extrapolation from code characters. This replaces the prior wrong behavior of suppressing parent results when a leaf exists. Added isolated test including `warehouse worker` → leaf 3372 + parent 3196 and DEPTH=0.

**Scope note:** Existing lexical prefix search is still the first retrieval stage and may miss titles such as `recruiter`; no ranking by CV duties, semantic search, AI selection or guaranteed global-best leaf is claimed. Parent results are navigation/Intake proposals, not verified employment facts. No CV/Uno integration, schema, migration, ESCO snapshot modification, React feature, or candidate persistence. Owner must pull Express main, run `npm run verify`, and execute real DB read-only retrieval check. Do not mark DONE until verification.


### Block A strict TypeScript repair (2026-10-10)

**VERIFY — owner re-run pending.** Owner's post-correction local `npm run verify` failed at TypeScript compilation with five errors: potentially undefined primary leaf, two potentially undefined parent labels, parent hit literal widening, and unchecked mock-call access in the test. Fixed on existing Express `main` in `src/esco/bridge/leaf-occupation-retrieval.ts` and `tests/esco-leaf-occupation-retrieval.test.ts`: explicit best-leaf guard, guarded `flatMap` parent label conversion with `LeafOccupationHit[]` return annotation, and optional mock-call access. No API, DB, migration, CV/Uno or policy changes. Prior green verify no longer applies to this revision. Owner must pull and run `npm run verify`; after green, run existing read-only `esco-retrieval-check.tmp.ts` for real PostgreSQL semantics. Do not mark DONE pending owner evidence.


### Block A — owner real PostgreSQL retrieval verification (2026-10-10)

**VERIFY (runtime graph traversal confirmed; full updated npm verification not evidenced in this report).** Owner executed `npx tsx .\\esco-retrieval-check.tmp.ts` against the local ACTIVE ESCO dataset after Block A correction and provided complete read-only output. Observed:
- DEPTH=0: `pharmacy assistant` -> 3231 only; `warehouse worker` and `warehouse order picker` -> leaf 3372 only; `ambassador` and `diplomat` -> leaf 1172 only; `embassy counsellor` -> 2846 only; `forklift operator` -> 3708 only; `recruiter` -> NO MATCH.
- DEPTH=1: leaf 3372 + immediate parent `warehouse worker` 3196 for both warehouse queries; 1172 + immediate parent `diplomat` 1258 for ambassador/diplomat; 2846 + 1258 for embassy counsellor; no parent for pharmacy assistant/forklift operator; `recruiter` remains NO MATCH.
- DEPTH=2: no further ancestors on the tested concepts, and results identical to DEPTH=1.
- Known caveats: existing prefix/alternative-label retrieval chooses ambassador as first leaf even for `diplomat`; no result for `recruiter`; first lexical leaf is **not** evidence of best CV occupation. No CV/Uno integration was performed. The owner's earlier green 181-test verification preceded the strict-TS repair, so require explicit post-repair `npm run verify` confirmation before marking whole Block A DONE.

The PostgreSQL read-only graph semantics of the updated implementation are now observed working. No data/migration changes. Await owner approval for Block B.


## ESCO CV → Uno Block B integration (2026-10-10)

**VERIFY — explicit owner-approved change on existing Express main; local compiler/tests and real CV result pending.** Scope: experimental `CandidateEscoAiResearchService` only, `src/dependencies.ts` injection, no DB/schema/migration/React or production intake changes. Introduced `EscoLeafOccupationRetrieval` into dev-only CV AI research while preserving OpenAI Uno interpretation and allowlisted selection, original CV draft, and existing skill proposal path. Replaced max-70 global union (unbounded lexical occupation prefixes plus FTS English description matches) with a bounded max-30 **leaf-only** candidate pool from up to 9 job/discovery queries plus up to 6 per-period fallback phrases, each using the standalone depth-limited graph retrieval. Uno sees **leaf concepts only**, with official English descriptions; after one allowlisted leaf selection, direct ancestor suggestions returned by the retrieval are appended to `experience.occupations` as additional unconfirmed proposals. Existing graph-related and separately proposed skills are retained, not automatically confirmed. Default max depth 1 through existing `ESCO_OCCUPATION_MAX_PARENT_DEPTH`; data unchanged.

**Important limitations/risk:** The first lexical leaf per search phrase is not guaranteed to be best by job duty semantics; `recruiter` previously returned no candidate in isolated prefix lookup. The existing Uno evidence-ranking orders the pool but can't recover candidates absent from it. Added ancestor suggestion appears as an occupation proposal, without separate parent-depth/approval flags in the current DTO. No changes to task contracts or React; UX may show both candidates indistinguishably until separately approved design work. Full regression needs owner local `npm run verify`, dev-only AI ↔ ESCO run on saved examples and compare outcomes across staffing, poultry packing, DAF, pharmacy, recruitment and dairy. Do not mark DONE or accuracy improved without evidence.


### Block B — test injection correction (2026-10-10)

**VERIFY, owner local retest pending.** Post-integration owner `npm run verify` failed in `tests/candidate-dev/esco-ai-research.test.ts` at three constructor call sites (TS2554: three arguments supplied, new `CandidateEscoAiResearchService` requires four). Updated all three test fixtures on Express `main` to inject a mocked read-only leaf retrieval; first fixture supplies one allowlisted leaf occupation and others return no leaf, retaining existing skill/no-active-dataset assertions. No application contract, DB, migration, React, ESCO snapshot or AI provider changes in this repair. Owner to pull `main` and run `npm run verify`; do not mark DONE before checks.


### Block B — whitelist-before-limit regression fix (2026-10-10)

**VERIFY.** Owner's local test run: 180/181 passed, one failure in `tests/candidate-dev/esco-ai-research.test.ts` where mocked Uno returned `[999999, 42]`. The Block B integration sliced the first model ID before checking the server-side allowlist, causing a valid second ID to be dropped. Corrected the existing Express `main` selection loop to deduplicate, **filter allowlisted IDs first**, then take one. The invalid ID cannot become an occupation; valid `42` is retained. No change to parent expansion, database, migration, ESCO source or React. Owner to `git pull --ff-only origin main` and `npm run verify`. Not verified locally by agent.


### ESCO Leaf ONLY database boundary (2026-10-10)

**VERIFY — owner approval and code on Express main; local tests and PostgreSQL behavior pending.** Corrected the owner-identified architecture violation: previous `EscoLeafOccupationRetrieval.search` queried broad occupation labels and only subsequently removed parent classes in TypeScript, allowing parent hits to consume the search LIMIT. The existing `EscoKnowledge.search` input now supports an optional `leafOnly` flag, and `PrismaEscoKnowledge.search` constrains occupation SQL with `NOT EXISTS` on `hb_esco_hierarchy_closure` for any `OCCUPATION_HIERARCHY` descendant with `hb_depth > 0`. This is evaluated in the ranked query **before** sorting and LIMIT. Normal search and skill callers default to unchanged behavior. The leaf retrieval always requests `leafOnly: true`, uses only the first returned terminal occupation for the dev Uno selection pool, and separately loads actual ancestor via `hb_descendant_concept_id`, bounded by configuration `ESCO_OCCUPATION_MAX_PARENT_DEPTH` (default 1) **only after leaf lookup**; ancestors are never retrieved as search candidates or offered to Uno selection. Existing graph data, Prisma schema, migration, React and persisted Candidate records remain unchanged. The leaf retrieval unit test now asserts leafOnly is sent and no pre-filtering ancestor query is executed.

**Explicit limitations:** lexical labels and official description-based ranking may still miss occupations; this change enforces candidate class but does not solve semantic recall. The non-leaf occupation graph is used only to identify excluded parents and load the direct ancestor of a selected leaf; no AI traversal of upper levels. Owner to pull Express main, run `npm run verify`, then verify live DB cases `warehouse worker` and `warehouse order picker` at depth zero and one. Status remains VERIFY pending owner evidence.


## SIMPLE PIPELINE — owner approved replacement (2026-10-10)

**VERIFY — implementation on Express main, local tests and timing/quality pending.** Owner explicitly rejected separate Uno work-context interpreter, extra query generation, hand-built lexical ranking, and pre-acceptance AI skills selection. The dev-only Candidate CV AI research service was simplified: per work period, deterministic title lookup plus PostgreSQL full-text official English DESCRIPTION lookup, both bounded to **leaf-only occupations before SQL LIMIT**; one Uno `candidate.esco.select` v2 call with the actual period title, description, and allowlisted 50-or-fewer leaf candidates; at most three real selected leaf IDs with server-side whitelist checks. No discovery/interpretation model call and no occupation ranking heuristic in active experimental path. Immediate ESCO parent suggestions (default depth 1) are obtained only after selecting leaves and not submitted to Uno. No ESCO skills are proposed/loaded before occupation acceptance. Draft CV parsing upstream remains separate and unchanged. Existing DTO retains empty `directSkills`/`relatedSkills` in the AI research response; the actual **post-acceptance skills-loading workflow and manual skill confirmation UI are not implemented by this change** and must be built in a separately approved task.

Updated files: `src/candidate-dev/esco-ai-research.ts`, `src/esco/knowledge.ts` (optional `leafOnly` on official description SQL lookup), `src/esco/bridge/leaf-occupation-retrieval.ts` (`parentsOfLeaf` for post-selection parent lookup), `tests/candidate-dev/esco-ai-research.test.ts` (one AI selection/allowlist/no preacceptance skills). Older bridge modules are left in the repository if needed by other consumers; not executed by this experiment. No migration, React, ESCO snapshot modification, or confirmed persistence. Known risk: description OR full-text can still return noise; benchmark real CV quality and latency. Owner needs `git pull --ff-only origin main`, `npm run verify`, then comparison with the two prior CV inputs. Do not mark DONE based on untested code.


**Simple pipeline refinement (same block, 2026-10-10):** Removed eager graph parent lookup during occupation candidate gathering. `CandidateEscoAiResearchService` now invokes `EscoKnowledge.search({leafOnly:true})` directly (up to 8 label hits per title segment), combines it with leaf-filtered description hits, makes only one Uno selection call per period, then invokes `EscoLeafOccupationRetrieval.parentsOfLeaf(id)` only for allowlisted selected Leaf IDs. Updated tests accordingly. All writes on Express main; owner `npm run verify` still required. No post-acceptance persistence is claimed.
