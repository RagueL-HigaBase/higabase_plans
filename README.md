# HigaBase Plans

**MANDATORY RULES:** [AGENT_RULES.md](AGENT_RULES.md). Read before any changes.

**CURRENT TASKS:** [TASKS.md](TASKS.md) → [React documentation](projects/react/) / [Express documentation](projects/express/).

**START HERE:** [Documentation Index](DOCUMENTATION_INDEX.md) → [Project State](PROJECT_STATE.md) → [System Map](docs/architecture/SYSTEM_MAP.md) → [Execution Plan](docs/operations/NEXT_STEPS.md). See [ADR-0001](docs/decisions/ADR-0001-DOCUMENTATION-AUTHORITY.md) for central authority. Current implementation repositories remain on `main` only.

## Documentation hierarchy

Use documentation in this order:

1. `SYSTEM_ARCHITECTURE.md` — cross-project system invariants and domain boundaries.
2. `ESCO_ARCHITECTURE.md` — shared ESCO knowledge, multilingual search/display, importer and consumer boundaries.
3. `CANDIDATE_ARCHITECTURE.md` — Candidate / Resume / Intake / ESCO / verification / component boundaries.
4. `UI_STANDARDS.md` — cross-project semantic UI roles and typography conventions.
5. `FINANCIAL_POLICY.md` — shared pricing direction, module boundaries, AI usage limits and integration policy.
6. Central `projects/react/PROJECT_RULES.md` or `projects/express/PROJECT_RULES.md` — rules specific to one implementation repository.
7. Central `projects/react/CURRENT_TASK.md` or `projects/express/CURRENT_TASK.md` — active temporary implementation scope.
8. Central `projects/react/PROJECT_STATE.md` or `projects/express/PROJECT_STATE.md` — implemented current state and known gaps.
9. Central project domain and production documents under `projects/<project>/docs/`.

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

## Documentation relocation (2026-10-08)

Project MD moved into `projects/react/` and `projects/express/` with original paths retained; implementation repositories retain only README redirects. New active tasks and project state updates belong in Plans, not duplicated local MD. Historical snapshots under `docs/repository-snapshots/` stay untouched.
