# HigaBase — Documentation Index

Updated: 2026-10-08. **This is the first document every new agent must read.**

## Mandatory reading order

1. [Mandatory agent rules](AGENT_RULES.md): main-only, exact task scope and explicit architecture approval.
2. [Cross-repository project state](PROJECT_STATE.md): current status, evidence, gaps and blockers.
3. [Architecture domain map](docs/architecture/SYSTEM_MAP.md): how the system fits together and which implementation owns each domain.
3. [System architecture](SYSTEM_ARCHITECTURE.md): binding identity, organization, permissions and data-flow invariants.
4. [Candidate architecture](CANDIDATE_ARCHITECTURE.md): global Candidate, Resume, consent, review and intake.
5. [ESCO architecture](ESCO_ARCHITECTURE.md): frozen ESCO dataset, graph, search and normalization.
6. [AI Core architecture](AI_CORE.md): provider-neutral contracts and data provenance.
7. [Execution plan](docs/operations/NEXT_STEPS.md): next verified gates and deferred work.
8. [Documentation governance](DOCUMENTATION_GOVERNANCE.md) and [ADR-0001](docs/decisions/ADR-0001-DOCUMENTATION-AUTHORITY.md).
9. Consult [UI Standards](UI_STANDARDS.md) and [Financial Policy](FINANCIAL_POLICY.md) when relevant.

Then inspect the relevant implementation repository's **main** branch, current code, schema, tests, local run instructions and any still-active CURRENT_TASK. Do not change branches.

## Repository responsibilities

| Repository | Role |
|---|---|
| [HigaBase_Plans](https://github.com/RagueL-HigaBase/higabase_plans) | Central documentation authority |
| [higa_systems_express](https://github.com/RagueL-HigaBase/higa_systems_express) | Current business API, independent ESCO domain, approved Candidate schema foundation |
| [higa_systems_react](https://github.com/RagueL-HigaBase/higa_systems_react) | Current Phoenix business frontend |
| [higa_systems_native](https://github.com/RagueL-HigaBase/higa_systems_native) | Candidate mobile shell |
| [higa-esco-extractor](https://github.com/RagueL-HigaBase/higa-esco-extractor) | Frozen ESCO source extraction |
| [higa_systems](https://github.com/RagueL-HigaBase/higa_systems) | Older combined System Core; not the current React/Express implementation pair |

## Archive and history

- [Express source snapshots](docs/repository-snapshots/express/)
- [React source snapshots](docs/repository-snapshots/react/)
- [Legacy and other clients](docs/repository-snapshots/)
- [Archive policy](docs/archive/README.md)

**Snapshots are historical evidence and are not maintained as current-state duplicates.** Original repository files have not been removed; the complete recursive Markdown inventory is still pending. Never claim that all files have been centralized until parity is checked.

## Agent guardrails

- **Only existing main**: never create a feature branch, switch away, or open PRs.
- Do not automatically merge/delete older branches; postpone their final cleanup until Candidate+ESCO closure.
- Proposed architecture ≠ implemented feature; green tests ≠ migrated local DB ≠ owner-reviewed UI.
- Do not expose experimental CV parsing/preview as production Candidate API.
- Canonical architecture is central; code/migrations/tests are evidence of actual implementation. Record discrepancies explicitly.
