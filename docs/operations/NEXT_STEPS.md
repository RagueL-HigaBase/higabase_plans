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
