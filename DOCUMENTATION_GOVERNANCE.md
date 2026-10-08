# Documentation Ownership and Governance

**Adopted documentation consolidation checkpoint: 2026-10-08.**

## Single-source-of-truth policy

- `SYSTEM_ARCHITECTURE.md`, `CANDIDATE_ARCHITECTURE.md`, `ESCO_ARCHITECTURE.md`, `AI_CORE.md`, `UI_STANDARDS.md`, `FINANCIAL_POLICY.md`: canonical cross-project decisions.
- Central `PROJECT_STATE.md`: consolidated *current* checkpoint, including evidence, blockers and next gate. No fiction about green local commands.
- Original implementation repository markdown: temporary historical/operational references while consolidation is verified; do not delete until complete comparison.
- `docs/repository-snapshots/`: verbatim-as-of-transfer source content with origin annotations. **Not updated as live documents**; preserve them to prevent lost architecture decisions.
- Code/schema/migrations/tests in repository `main`: implementation evidence, not substitutes for approved architecture. If conflict exists, log it and reconcile explicitly.

## Mandatory agent startup checklist

1. Read central index and cross-system state.
2. Read relevant central domain architectures completely.
3. Read the live main branch and any still-active local `CURRENT_TASK.md`.
4. Confirm scope and source of authority, record outstanding contradictions.
5. Check status/worktree before edits. Use `main` only. Never create a temporary branch.
6. Work in small approved blocks; verify; update state with exact evidence.

## Change ownership

Every architectural decision is edited first centrally. Implementation-specific run commands, env and executable tests can remain in repo README/AGENTS or thin pointers. Avoid copying the same domain rules across repos. When code is changed, central `PROJECT_STATE.md` is updated only to factual verified state; current work stays explicitly `IN PROGRESS` or `VERIFY`.

## Consolidation safeguards

The first migration step is **copy without deletion**. A future review must enumerate all repository markdown, compare source and archive, resolve contradictions and then replace local long-form descriptions with short pointers. Do not delete original domain and production docs simply because a snapshot exists. This repository is the intended destination, but relocation completeness is **not yet certified**.

## Status vocabulary

`PROPOSED` — not approved; `READY` — scoped and approved; `IN PROGRESS` — changing code; `VERIFY` — code exists but needed verification pending; `DONE` — owner-confirmed verified; `BLOCKED` — cannot proceed safely. Always label historical records with their date.
