# HigaBase Plans
> **Agent entry point (2026-10-08):** Start with [DOCUMENTATION_INDEX.md](DOCUMENTATION_INDEX.md), then [PROJECT_STATE.md](PROJECT_STATE.md) and [DOCUMENTATION_GOVERNANCE.md](DOCUMENTATION_GOVERNANCE.md). Implementation documentation is being consolidated into `docs/repository-snapshots/` as source evidence. These snapshots are not competing architectural authorities.


Central architecture and planning repository for the HigaBase ecosystem.

This repository owns cross-project architectural rules that must remain consistent across the web frontend, backend and future clients.

## Documentation hierarchy

Use documentation in this order:

1. `SYSTEM_ARCHITECTURE.md` — cross-project system invariants and domain boundaries.
2. `ESCO_ARCHITECTURE.md` — shared ESCO knowledge, multilingual search/display, importer and consumer boundaries.
3. `CANDIDATE_ARCHITECTURE.md` — Candidate / Resume / Intake / ESCO / verification / component boundaries.
4. `UI_STANDARDS.md` — cross-project semantic UI roles and typography conventions.
5. `FINANCIAL_POLICY.md` — shared pricing direction, module boundaries, AI usage limits and integration policy.
6. Repository-local `PROJECT_RULES.md` — rules specific to one implementation repository.
7. Repository-local `CURRENT_TASK.md` — active temporary implementation scope.
8. Repository-local `PROJECT_STATE.md` — implemented current state and known gaps.
9. Repository-local domain and production documents.

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
- cross-project contracts;
- cross-client semantic UI conventions;
- Candidate / Resume / intake / verification architecture;
- shared ESCO knowledge and multilingual semantic-reference architecture;
- shared commercial/pricing principles and variable-cost automation boundaries.

Do not use this repository for:
- implementation-specific source code;
- current build status;
- temporary TODOs;
- framework-specific styling or component rules;
- duplicated copies of repository-local state.

Last reviewed: 2026-10-06
