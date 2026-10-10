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
