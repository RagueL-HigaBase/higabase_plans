# Documentation Ownership and Governance

**Adopted documentation consolidation checkpoint: 2026-10-08.**

## Single-source-of-truth policy

- `SYSTEM_ARCHITECTURE.md`, `CANDIDATE_ARCHITECTURE.md`, `ESCO_ARCHITECTURE.md`, `AI_CORE.md`, `UI_STANDARDS.md`, `FINANCIAL_POLICY.md`: canonical cross-project decisions.
- Central `PROJECT_STATE.md`: consolidated *current* checkpoint, including evidence, blockers and next gate. No fiction about green local commands.
- Central `projects/react/` and `projects/express/` are the authoritative live per-project MD locations. Original project MD were copied with SHA parity before removing duplicates from source repositories; source README redirects remain.
- `docs/repository-snapshots/`: verbatim-as-of-transfer source content with origin annotations. **Not updated as live documents**; preserve them to prevent lost architecture decisions.
- Code/schema/migrations/tests in repository `main`: implementation evidence, not substitutes for approved architecture. If conflict exists, log it and reconcile explicitly.

## Mandatory agent startup checklist

1. Read central index and cross-system state.
2. Read relevant central domain architectures completely.
3. Read the live implementation main branch and its **central** `projects/<project>/CURRENT_TASK.md`.
4. Confirm scope and source of authority, record outstanding contradictions.
5. Check status/worktree before edits. Use `main` only. Never create a temporary branch.
6. Work in small approved blocks; verify; update state with exact evidence.

## Change ownership

Every architectural decision is edited first centrally. Implementation-specific run commands, env and executable tests can remain in repo README/AGENTS or thin pointers. Avoid copying the same domain rules across repos. When code is changed, central `PROJECT_STATE.md` is updated only to factual verified state; current work stays explicitly `IN PROGRESS` or `VERIFY`.

## Consolidation safeguards

On 2026-10-08, a full recursive inventory of React/Express Markdown found 24+23 files. Copies under `projects/` were checked against source blob SHAs before removal. The older `docs/repository-snapshots/` retains historical source evidence. Relative-link reconciliation and semantic contradictions are separate follow-up documentation work; byte parity does not certify content accuracy.

## Status vocabulary

`PROPOSED` — not approved; `READY` — scoped and approved; `IN PROGRESS` — changing code; `VERIFY` — code exists but needed verification pending; `DONE` — owner-confirmed verified; `BLOCKED` — cannot proceed safely. Always label historical records with their date.

## Mandatory execution rules

The non-negotiable owner instructions are in [AGENT_RULES.md](AGENT_RULES.md). These supersede any permissive suggestion elsewhere: existing main only, exactly requested scope, no unsolicited implementation, owner approval before architectural changes. Suspected merge regressions are tracked in [PROJECT_STATE.md](PROJECT_STATE.md) and [docs/operations/NEXT_STEPS.md](docs/operations/NEXT_STEPS.md).

## Current task authority

`TASKS.md` is the sole central cross-project active-work entry point. Per-project details live in `projects/react/CURRENT_TASK.md` and `projects/express/CURRENT_TASK.md`. Documentation updates in implementation blocks must target these central paths; source README files provide pointers only.
