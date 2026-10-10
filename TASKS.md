# HigaBase — Active Tasks

Updated: 2026-10-08
Status: CENTRAL WORK QUEUE (not automatic implementation approval)
Authority: [AGENT_RULES.md](AGENT_RULES.md), [PROJECT_STATE.md](PROJECT_STATE.md), [NEXT_STEPS.md](docs/operations/NEXT_STEPS.md).

## Rules

- This is the **single entry point for current cross-project work**. Read this file before implementation, then the relevant project task and domain specifications under `projects/`.
- Only existing `main`. Do not create branches or PRs.
- A listed priority is not approval to execute it. Owner explicitly approves each narrow change.
- Each approved code block updates its central project task MD **in the same block**; update central domain/state MD only for verified durable facts.
- Distinguish code / CI / local DB / browser tests / owner acceptance. `VERIFY` is never `DONE`.
- Preserve original history in `projects/react` and `projects/express`; do not reinterpret their imported 2026-10-08 copies as freshly verified states.

## Current work board

| Priority | Task | Scope / acceptance | Status | Project |
| --- | --- | --- | --- | --- |
| P0 | Candidate Core migration reconciliation | Inspect local DB migration history vs `20261008192000_candidate_core_foundation`, stray `20261006145754_candidate`, and ESCO prefix index. No reset, deletion or blind deploy. | BLOCKED / owner DB inspection | [Express](projects/express/CURRENT_TASK.md) |
| P0 | Candidate Core schema-only foundation | Confirm committed Prisma schema with owner-run generate, verify, safely reconciled migration and DB status; no Candidate API. | VERIFY | [Express](projects/express/docs/DATABASE.md) |
| P1 | React workspace restoration acceptance | Local `npm run verify` and browser check: PIN, email verification, Organization/Locations, Candidate Prototype and recruitment UI. | VERIFY | [React](projects/react/docs/planning/WORKSPACE_RECOVERY_2026_10_08.md) |
| P1 | CV file preview acceptance | Local synthetic PDF/DOCX browser flow via development-only, authenticated preview; no production storage or Candidate row creation. | VERIFY | [React](projects/react/CURRENT_TASK.md), [Express](projects/express/CURRENT_TASK.md) |
| P1 | Recruitment Candidates Phoenix scaffold | Verify listing UI, sorting, filters and Add Candidate informational/preview flow within approved scope; no Candidate list API. | VERIFY | [React](projects/react/CURRENT_TASK.md) |
| P1 | AI parser benchmark | Review available measured evidence, remaining hard fixtures and provider-specific results before closing. | IN PROGRESS | [Express](projects/express/CURRENT_TASK.md) |
| P2 | Candidate relations / consent / audit | Design first; explicit owner approval needed before schema/authorization/API changes. | PROPOSED | [Candidate Architecture](CANDIDATE_ARCHITECTURE.md) |
| P2 | Source artifact, Resume Core, ESCO proposals, screening | Follow approved sequential gates with source preservation and strict authorization; no automatic kickoff. | PROPOSED | [Execution Plan](docs/operations/NEXT_STEPS.md) |
| Deferred | Functional/regression audit from Sign In, then SonarQube | After approved Candidate + ESCO blocks, unless owner explicitly changes priority. | PROPOSED | [Project State](PROJECT_STATE.md) |
| Deferred | Historical branch audit and cleanup | No branch deletion until owner-approved evidence-based comparison. | BLOCKED | [Agent Rules](AGENT_RULES.md) |

## Documentation relocation

- **Live central document homes:** [React](projects/react/) and [Express](projects/express/). The imported files retain their former relative paths as of 2026-10-08.
- Central architectural authority stays in root cross-domain specifications (`SYSTEM_ARCHITECTURE.md`, `CANDIDATE_ARCHITECTURE.md`, `ESCO_ARCHITECTURE.md`, `AI_CORE.md`).
- `docs/repository-snapshots/` is older historical evidence only.
- Source repos should contain code plus a small README redirect. Newly updated implementation documentation belongs **only in Plans**, in its respective project folder. Change reports must cite the exact central MD path.
- If GitHub main and user local worktree differ, do not claim this board proves the local state; inspect it first.

## Candidate development checkpoint — 2026-10-09

**[End-of-day evidence and next-session plan](docs/operations/CANDIDATE_DEV_CHECKPOINT_2026_10_09.md)** is the current factual checkpoint for the experimental Candidate pipeline (supersedes the **stale candidate-only task statuses above**, not the unrelated work queue). Owner-verified: development CV parsing → Candidate/Contact/ResumeDraft transaction → Recruitment Candidates list → candidate detail; Express 35 files/150 tests, 16 DB migrations up to date, React browser presentation verified and local verify reported green. Reset command found four candidates and safely cancelled; **actual deletion not tested**. Next proposed work: dev-only CV rate limit 30/15min and stale Preview banner fix, then owner-decided controlled Reset and CV batch audit. **Proposals are not implementation approval.** Existing `main` only.

## Development CV hardening — 2026-10-10

**VERIFY / owner checks pending:** Express `main` shared development CV limit set to 30/15min, test for 31st request added; React `main` clears old Preview before Save. See `projects/express/CURRENT_TASK.md` and `projects/react/CURRENT_TASK.md`. No migrations; no reset executed. Next: owner `git pull` and `npm run verify` in both repositories; browser check before DONE. The existing ESCO Knowledge implementation and AI benchmark (F1, latency, token-based OpenAI cost estimation) were confirmed present in source; do **not** confuse absence of an observed ESCO call on the current CV Preview/Save path with absence of ESCO or benchmark features. Dedicated integration audit remains proposed.

## Candidate ESCO visual research — 2026-10-10

**VERIFY, owner checks pending:** Experimental read-only ESCO proposals attached to dev Candidate Details; AI Extraction remains separate; ESCO Occupations and ESCO Skills shown independently. Related skills retain ESSENTIAL/OPTIONAL, are not Candidate-confirmed. Current resolver intentionally only uses exact ESCO preferred/alternative term equality to avoid mislabeling prefix hits; semantic inference is not implemented. Source code React/Express updated on existing main, focused route test added. No migration, data reset, AI provider calls, persist or auto-approval. See central project CURRENT_TASK files. Await local Express/React `npm run verify` and owner visual assessment, especially coverage of diverse CVs.

## Candidate ESCO Timeline prototype (2026-10-10)

**VERIFY**. Work Experience is the read-only research unit, with period-scoped ESCO Occupations, evidence-linked Direct Skills, and separately collapsed graph Related Skills. Unassigned whole-CV ESCO skill matches remain separate, preserving provenance; CV-wide AI Skills and Tools remain unchanged. No Canonical ESCO/Candidate changes, DB migrations, persistence, confirmation, AI rerun or new semantic model. New research mapper tests plus revised detail route test. Owner should pull React and Express main, run both npm run verify, restart dev services, view CV examples, and report results. Historical frontend/global list superseded in the experiment, not in the canonical architecture.

## ESCO Timeline TS strict access hotfix — 2026-10-10

Status **VERIFY (owner rerun required)**. Fixed 16 owner-reported TS strict/noUncheckedIndexedAccess errors (4 in `src/candidate-dev/esco-research.ts`, 12 in `tests/candidate-dev/esco-research.test.ts`): runtime guards for indexed work-period records; explicit non-null assertions only for known fixture entries in tests. No ESCO model or graph changes, migrations, new AI logic, or frontend change. Owner to `git pull` Express main and `npm run verify`; no claim of green until owner confirms.


## 2026-10-10 CV intake and AI–ESCO research

2026-10-10: Dev CV one-step UX + cost disclosure committed in React main. Save Test Candidate is one action after choosing CV; Preview removed; drawer closes when the existing server POST finishes. THIS IS NOT asynchronous backend/background processing. Candidate sidebar currently says `AI cost: Not recorded` (no fake $0.00). Research finding: AI provider supports only resume.parse, no AI-ESCO selection yet; next block to implement typed AI↔ESCO per-work-period selection and accurate per-operation usage/cost. Owner React verify and browser checks pending.


## AI ↔ ESCO Research V1 (2026-10-10)

2026-10-10 AI ↔ ESCO experiment IMPLEMENTED IN MAIN, VERIFY pending local checks. OpenAI provider supports two new bounded tasks; dev-only authenticated POST classification on existing saved ResumeDraftV1; per-work experience whitelist of ESCO Occupation IDs + select relevant Skill IDs from actual graph edges. No persistence, no new tables, no ESCO architectural change, no intake confirmations, no cost changes. UI explicit run button prevents repeated paid calls on page GET; on success per-work time blocks show AI selections. Known limitations: English lexical query candidate recall, high potential latency with up to 8 periods, AI may choose none, maximum 3 experimental requests per 15 min, OpenAI-only, no durable job. Verify and inspect CV locally before any further changes.


## ESCO concept tooltip and cost research

2026-10-10 ESCO Popovers + iCosts checkpoint — **VERIFY, owner npm run verify/browser pending**. Experimental Candidate Details now uses shared `EscoConceptBadge` with React-Bootstrap Popover on hover/focus for all ESCO occupation/skill/related/unassigned badges. Fetches official description lazily through new read-only `GET /api/esco/concepts/:id/description` using active dataset's EN/NL DESCRIPTION/DEFINITION/SCOPE_NOTE, cached per concept in React. Displays clear fallback if description absent; never uses whole CV text as description. Existing graph and migrations unchanged; added endpoint route test. Cost: no feature change; current historic Candidate says `Not recorded`; accurate per-candidate iCosts require collecting operation model + actual input/output tokens, pricing tariff/version and operation ID at execution; Billing can reconcile totals, cannot reliably allocate historical charges to candidate. No API/admin key requested. Visual and real description content needs owner check.


## Occupation-first ESCO research UI checkpoint — 2026-10-10

Status **VERIFY — owner npm run verify/browser pending**. Removed unreachable `Open ESCO concept` link from read-only badge popover; official ESCO description remains on hover/focus. Removed expanded Related ESCO Skills details blocks from experimental Candidate Details UI only. Related graph remains in backend `candidate.esco.experiences[].relatedSkills` and unchanged ESCO tables for future graph-distance/similarity research; did not delete data or change ESCO architecture. Main view emphasizes work-period occupation proposals and relevant AI skill proposals. **Do not yet invent match percentages, mark AI proposals Confirmed, or claim deterministic repeatability**. Next research candidate occupation recall and rerun variation: evaluate leaf occupation concepts vs ISCO 10 major groups (taxonomy, not 10 ESCO professions), generate controlled candidate set, AI selection evidence and hierarchy distance, then propose a scoring contract before implementation. No semantic-matching backend changes in this checkpoint.


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
