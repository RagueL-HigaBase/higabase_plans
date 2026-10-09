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
