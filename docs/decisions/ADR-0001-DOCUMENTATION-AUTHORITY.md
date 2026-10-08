# ADR-0001 — Single Documentation Authority

Date: 2026-10-08
Status: ADOPTED (consolidation incomplete)

## Context

Cross-repository Markdown repeated architecture, current tasks, production history and implementation status. Agents switched among stale versions and feature branches, leading to conflicting assumptions.

## Decision

1. `HigaBase_Plans/main` is the central system-wide architecture authority.
2. Central `PROJECT_STATE.md` is the consolidated implementation **checkpoint**, updated with sourced verification. Current tasks must clearly show whether they are READY, IN PROGRESS, VERIFY, BLOCKED or DONE.
3. Active code lives in the existing `main` of React/Express. No new Git branches or PRs.
4. Implementation repository README/AGENTS may retain environment commands, tool-specific runbooks and minimal links, but cannot redefine cross-project invariants.
5. Old documents are copied as **historical source snapshots**, then reconciled against central docs and current code. The snapshots do not become parallel authoritative states. No original deletion until parity is verified.
6. No forced migration/DB modification is implied by documentation migration.

## Consequences

- New agents read `DOCUMENTATION_INDEX.md` first, then state, architecture and task.
- Architecture decisions change centrally once; consumers link instead of copying.
- Preserve closure histories with dates; distinguish actual tests from old owner-reported results.
- A follow-up recursive inventory and link audit are mandatory before calling consolidation complete.
