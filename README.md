# HigaBase Plans

Central architecture and planning repository for the HigaBase ecosystem.

This repository owns cross-project architectural rules that must remain consistent across the web frontend, backend and future clients.

## Documentation hierarchy

Use documentation in this order:

1. `SYSTEM_ARCHITECTURE.md` — cross-project system invariants and domain boundaries.
2. Repository-local `PROJECT_RULES.md` — rules specific to one implementation repository.
3. Repository-local `CURRENT_TASK.md` — active temporary implementation scope.
4. Repository-local `PROJECT_STATE.md` — implemented current state and known gaps.
5. Repository-local domain and production documents.

A repository-local rule may refine a central rule for implementation details, but it must not contradict a central system invariant.

## Active repositories

- Backend/API: `RagueL-HigaBase/higa_systems_express`
- Web frontend: `RagueL-HigaBase/higa_systems_react`
- Candidate mobile client: `RagueL-HigaBase/higa_systems_native`
- ESCO extraction/normalization: `RagueL-HigaBase/higa-esco-extractor`

## Scope

Keep this repository conceptual.

It should contain:
- domain boundaries;
- identity rules;
- organization/location ownership rules;
- invitation lifecycle rules;
- future permission architecture;
- audit/history invariants;
- cross-project contracts.

Do not use this repository for:
- implementation-specific source code;
- current build status;
- temporary TODOs;
- framework-specific styling or component rules;
- duplicated copies of repository-local state.

Last reviewed: 2026-10-04
