# HigaBase Documentation — Start Here

**Central documentation authority.** Reviewed 2026-10-08. Read this before changing any repository.

## Reading order (mandatory for agents)

1. [PROJECT_STATE.md](PROJECT_STATE.md) — one cross-repository operational checkpoint, current/verified/pending/blocked work.
2. [SYSTEM_ARCHITECTURE.md](SYSTEM_ARCHITECTURE.md) — binding platform-wide identity, organization, permission and lifecycle invariants.
3. [CANDIDATE_ARCHITECTURE.md](CANDIDATE_ARCHITECTURE.md) — Candidate/Resume/consent/intake boundaries.
4. [ESCO_ARCHITECTURE.md](ESCO_ARCHITECTURE.md) — frozen ESCO reference model, import and semantic resolution.
5. [AI_CORE.md](AI_CORE.md) — provider-independent AI execution and provenance.
6. [UI_STANDARDS.md](UI_STANDARDS.md) and [FINANCIAL_POLICY.md](FINANCIAL_POLICY.md) as relevant.
7. [DOCUMENTATION_GOVERNANCE.md](DOCUMENTATION_GOVERNANCE.md) — document ownership, change procedure, snapshot limitations.
8. Only then inspect the relevant implementation repository **main** branch, its runtime contracts/tests/schema, and its minimal local agent instructions.

## Actual repositories

- [HigaBase_Plans](https://github.com/RagueL-HigaBase/higabase_plans): central cross-system architecture and authoritative project checkpoint.
- [higa_systems_express](https://github.com/RagueL-HigaBase/higa_systems_express): active Express/Prisma API.
- [higa_systems_react](https://github.com/RagueL-HigaBase/higa_systems_react): active Phoenix React business web client.
- [higa_systems_native](https://github.com/RagueL-HigaBase/higa_systems_native): early Candidate mobile shell.
- [higa-esco-extractor](https://github.com/RagueL-HigaBase/higa-esco-extractor): ESCO source extraction.
- [higa_systems](https://github.com/RagueL-HigaBase/higa_systems): older combined API/web System Core repository; **not** the current business client/server pair. Do not confuse it with HigaBase_Plans.

## Documentation consolidation

Original React/Express markdown has been **copied, not removed**, to [docs/repository-snapshots/](docs/repository-snapshots/). These copies are **historical, immutable-style source snapshots**, not another current-state authority. The central documents above determine architecture; source code and tests on main determine observed implementation. Review snapshot evidence before deprecating originals. Do not turn copied CURRENT_TASK files into competing active tasks.

## What not to assume

- Implemented code ≠ deployed/database-applied ≠ owner-verified.
- A benchmark or experimental dev CV preview ≠ production Candidate intake.
- A Git branch containing ahead commits ≠ missing main functionality (squash merges are common).
- The local PostgreSQL state is not accessible through the GitHub connector.
- Never invent progress, close VERIFY tasks, or overwrite history without evidence.
