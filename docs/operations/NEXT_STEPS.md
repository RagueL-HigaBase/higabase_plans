# Current Execution Plan — 2026-10-08

Central live task queue: [TASKS](../../TASKS.md). Source of truth for project-wide checkpoint: [PROJECT_STATE](../../PROJECT_STATE.md). This document defines ordering, not independent task approvals.

## Step 0 — Documentation consolidation

- [x] Introduce central agent index and state checkpoint.
- [x] Archive initial Express/React implementation documentation without destroying originals.
- [x] Define system map and documentation governance.
- [ ] Complete **recursive inventory** of every relevant MD in all repositories and compare archive copies; unknown or unlisted documents remain out of scope for a completeness claim.
- [ ] Reconcile every local domain/production/planning record with canonical architecture, code and migration state.
- [ ] Replace long-form duplicated local narratives with short pointers **only after** content parity and owner review.
- [ ] Validate cross-repo Markdown links and update stale references.

## Step 1 — Stabilize existing main

- [x] Express owner local main synchronized on 2026-10-08; Prisma/client/typecheck/build and 141/141 tests passed.
- [ ] Express local migration discrepancy: applied stray `20261006145754_candidate` vs committed `20261008192000_candidate_core_foundation`; verify DB index and applied SQL before repair.
- [ ] React local main status and UI recovery verification (PIN/email/Organization/Locations/Candidate Prototype).
- [ ] Legacy combined `higa_systems` classification before retirement decisions.

## Step 2 — Candidate + ESCO approved progression

1. Confirm schema-only Candidate Core database foundation and migration.
2. Design/approve Candidate–Organization relation, access and consent boundary with audit.
3. Implement SourceArtifact/provenance-safe CV intake; do not reuse development routes as production APIs.
4. Implement Resume Core and validated AI draft bridge with preserved source text.
5. Implement ESCO occupation/skill mapping as suggestions tied to real EscoConcept.
6. Implement candidate verification and authorized frontend end-to-end.

## Deferred

- Existing non-main branch deletion until Candidate + ESCO architecture is closed; assess squash-merged equivalents and unique changes before deletion.
- Advanced permission matrices, automatic model routing, matching weights, mobile integration and speculative UI are not currently authorized.

Each block updates PROJECT_STATE with the exact Git/main revision and measured tests. Status VERIFY is not DONE.

## Deferred: full regression audit and SonarQube (OPEN)

After finishing the currently approved Candidate + ESCO blocks, inspect the live web behavior **from Sign In onward**; compare React, Express and PostgreSQL state against central architecture and previously verified UI contracts. The owner reported possible regressions associated with historical branch merges. Treat suspected lost data as unverified until traced. Log each mismatch as an isolated defect with reproduction, expected/actual behavior, affected commit/files, impact and verification. Repair only an explicitly approved defect. Then run SonarQube and fix reported issues without changing intended behavior. Keep statuses VERIFY until owner confirms. Do not start this work implicitly as part of Candidate tasks.

## Centralized documentation checkpoint — 2026-10-08

The 24 React and 23 Express Markdown files were copied into `projects/react/` and `projects/express/` with source blob-SHA parity; project MD are maintained centrally going forward. Track live priorities in [TASKS.md](../../TASKS.md). Semantic/link consolidation remains a separate documentation-only follow-up.

## Next session — Candidate Dev Pipeline (checkpoint 2026-10-09)

Read [the factual checkpoint](CANDIDATE_DEV_CHECKPOINT_2026_10_09.md) **before** acting on older Candidate steps above. The owner has a locally working development CV → AI → PostgreSQL → Recruitment table → Candidate Details vertical slice. Tomorrow's proposed, individually approval-gated order: (1) dev-only CV upload rate limit 5→30 per 15 minutes and React stale success-message fix; (2) optional, **explicitly owner-confirmed** experimental Reset deletion check; (3) diverse consented CV batch evaluation; (4) research AI extraction vs interpretation vs ESCO occupation/skills mapping. No production authorization, ESCO integration or visual redesign is implied.

## Progress — 2026-10-10 (VERIFY)

Dev-CV-Hardening-1 implementation committed to Express and React **main**; owner verification outstanding. The first source audit found `src/esco/knowledge.ts` / `routes.ts`, runtime/provider-neutral AI tasks, `src/ai/benchmark/resume-cli.ts` and `openai-benchmark-pricing.ts` already implemented; cost estimates need usage and configured model pricing. This is **not** evidence that the active CV intake currently invokes ESCO. Avoid reimplementation or speculative architecture work. Next owner checks: pull/verify each repo, browser retest; optional Reset only by explicit owner confirmation, then diverse CV assessment.

## Immediate verification: ESCO timeline research (2026-10-10)

Pull `main` in Express and React, run `npm run verify` in both and inspect saved Candidate Details. Check chronology (including undated source periods), per-row source/ESCO occupations/direct skills, collapsed ESSENTIAL/OPTIONAL graph relations, unassigned CV-wide matched skills, missing ESCO candidate empty states and existing AI Skills/Tools. Report mistakes before implementing any semantic matching, AI per-work-experience proposal generation or confirmation colors. This is a disposable read-only experiment, not production architecture. Central CURRENT_TASK and TASKS are authoritative tracking.


## 2026-10-10 CV intake and AI–ESCO research

2026-10-10 NEXT: (1) React main git pull/npm run verify; verify dev upload select file + Save closes on successful server response, and invalid requests show errors. (2) AI↔ESCO classification remains NOT IMPLEMENTED; inspect provider/task contract to implement bounded AI proposals against real ESCO IDs per work-history item, without altering ESCO canonical tables. (3) Before financial cost display, capture actual token usage/model and price version per ingestion operation; past intakes cannot be priced as actual billable operations. Async upload/background state transition requires server-side durable job design, not merely closing dialog early.


## AI ↔ ESCO Research V1 (2026-10-10)

2026-10-10 AI ESCO research checkpoint: pull Express and React main; run npm run verify in both. Reload saved Candidate Details, click 'Run AI ↔ ESCO research' once, wait for response, inspect per-period ESCO Occupations and AI Proposed Skills; Related Skills expand separately. Check no unnecessary requests on GET, access enforcement, invalid IDs rejected, no DB writes. Review first CV result and errors before tuning candidate recall or semantic mapping. Costs remain Not recorded and untouched. React upload is still synchronous save-and-close on response; async queue is future work. Do not change canonical ESCO schema.


## ESCO concept tooltip and cost research

2026-10-10 ESCO Popovers + iCosts checkpoint — **VERIFY, owner npm run verify/browser pending**. Experimental Candidate Details now uses shared `EscoConceptBadge` with React-Bootstrap Popover on hover/focus for all ESCO occupation/skill/related/unassigned badges. Fetches official description lazily through new read-only `GET /api/esco/concepts/:id/description` using active dataset's EN/NL DESCRIPTION/DEFINITION/SCOPE_NOTE, cached per concept in React. Displays clear fallback if description absent; never uses whole CV text as description. Existing graph and migrations unchanged; added endpoint route test. Cost: no feature change; current historic Candidate says `Not recorded`; accurate per-candidate iCosts require collecting operation model + actual input/output tokens, pricing tariff/version and operation ID at execution; Billing can reconcile totals, cannot reliably allocate historical charges to candidate. No API/admin key requested. Visual and real description content needs owner check.


## Occupation-first ESCO research UI checkpoint — 2026-10-10

Status **VERIFY — owner npm run verify/browser pending**. Removed unreachable `Open ESCO concept` link from read-only badge popover; official ESCO description remains on hover/focus. Removed expanded Related ESCO Skills details blocks from experimental Candidate Details UI only. Related graph remains in backend `candidate.esco.experiences[].relatedSkills` and unchanged ESCO tables for future graph-distance/similarity research; did not delete data or change ESCO architecture. Main view emphasizes work-period occupation proposals and relevant AI skill proposals. **Do not yet invent match percentages, mark AI proposals Confirmed, or claim deterministic repeatability**. Next research candidate occupation recall and rerun variation: evaluate leaf occupation concepts vs ISCO 10 major groups (taxonomy, not 10 ESCO professions), generate controlled candidate set, AI selection evidence and hierarchy distance, then propose a scoring contract before implementation. No semantic-matching backend changes in this checkpoint.


## Occupation Discovery V2 research (2026-10-10)

2026-10-10 Occupation Discovery V2 + skill discovery experiment — **VERIFY, local tests and real OpenAI run pending**. In Express main, added OpenAI task `candidate.esco.discover@1`: for each existing Work Experience the model proposes bounded English occupation lookup phrases and concrete skill lookup phrases from title/duties, no fabricated ESCO IDs. Service queries existing `EscoKnowledge.search` for Preferred/Alternative labels (prefix search) using complete phrases rather than blindly truncated 5-letter tokens, builds max-70 real occupation candidates, retains existing `candidate.esco.select@1` strict allowlist, then searches independent SKILL concepts as well as related skills and uses `candidate.esco.skills.select@1` to propose up to 12 real skill IDs. An experience may thus have skill proposals even if occupation discovery found none; everything is per sourceIndex and ephemeral. Candidate UI unchanged, existing proposed badge and official descriptions. No DB migrations, writes, ESCO graph/schema changes, human confirmed state, iCosts changes, or autosave. English prefix may still miss synonyms that do not begin with a proposed query, and results may vary per model run. Explicit button still causes paid and synchronous work; rate limit 3 per 15 min. Unit tests updated and new missing-occupation/skill scenario added; **owner must git pull Express main and npm run verify** and then inspect results.


## Context Verification V1 — 2026-10-10

2026-10-10 CONTEXT-VERIFICATION-V1 — **VERIFY, owner local tests pending**. Experimental Express AI/ESCO occupation selection now loads EN DESCRIPTION (fallback DEFINITION) from active ESCO concept versions for up to 70 discovered occupation candidates; passes preferred label, actual matched Preferred/Alternative term, official description (<=1100 chars) and actual Work Experience title/duties to existing typed candidate.esco.select task. OpenAI instructions prioritize documented duties, avoid unsupported manager/operator/supervisor seniority, and reject unsuitable occupations. Server validates selected ID against verified active-dataset occupation concepts; no new tables, migrations, changes to canonical ESCO data, persisted classification, cost accounting or React page. Existing per-period boundaries maintained. Test fixture now covers description+matchedTerm provided to AI selector. Important: no live AI accuracy claim until owner runs Express npm run verify and compares saved CV. Caveat: upstream prefix recall and missing official descriptions remain possible limitations.


## Occupation Discovery instruction experiment and diagnostics — 2026-10-10

Status **VERIFY — owner local npm run verify and live browser run pending**. User showed CV: Warehouse Employee → Warehouse worker, Electromontage → Electrician; Manager-consultant with retail sales/customer advice/payment tasks → no occupation. Changed only OpenAI discovery prompt to prioritize documented duties, realistic Preferred/Alternative occupation search phrases and avoid unsupported managerial seniority; no canonical ESCO mutations. Experimental read-only AI response adds per-work-history `diagnostics` with bounded occupationQueries, retrievedOccupations (real concept ID, label, matchedTerm) and selectedOccupationIds; React CandidateDetails displays them only inside collapsed Research Diagnostics for that same period. Added regression assertions for fixture timeline isolation. Nothing is persisted; results remain ephemeral and paid calls remain explicit. This is observability, NOT guaranteed improvement; owner to pull BOTH Express and React main, run both `npm run verify`, restart services and test one CV. Do not infer exact reason for a failed match without diagnostics.


## Esco Intelligence Bridge V1 — 2026-10-10

**VERIFY; owner local checks pending.** Added `src/esco/bridge/work-context.ts` as internal typed, read-only intermediary between Uno AI and existing ESCO graph. Existing `candidate.esco.discover@1` now emits **structured interpretation of one work period** (`coreActivities`, `responsibilities`, `equipment`, `workingConditions`, `evidenceGaps`) together with bounded occupation/skill ESCO lookup phrases, avoiding another model call. Work context is stored **only in ephemeral research API response** (`candidate.esco.experiences[].interpretation`) and displayed in collapsed `Uno Interpretation` for each Work Experience separately, alongside existing Research Diagnostics/ESCO suggestions. No DB/schema/migrations, ESCO alterations, auto confirmations, persistent AI decisions, billing changes or changes to candidate upload. Updated test mocks for typed discovery output and assertion on context; all test/build results **unverified pending owner `git pull` and `npm run verify` in BOTH Express and React**. Isolation still limits processing to first 8 work periods as previously. This is an observability/interpretation foundation, not proof of improved matching or semantic search. Compare interpretation, retrieved candidates and final ESCO selections on same CV to isolate failure source.
