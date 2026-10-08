# ESCO Knowledge Graph Foundation — Verified Closure

Status: **CLOSED**
Verified: **2026-10-06**

Canonical domain document:
- `docs/ESCO.md`.

## Closed scope

This block established the PostgreSQL/Prisma persistence foundation for the HigaBase ESCO Knowledge Graph.

Implemented:
- versioned `EscoDataset`;
- stable URI-backed `EscoConcept`;
- dataset-scoped `EscoConceptVersion`;
- multilingual `EscoLabel` and `EscoText`;
- source-preserving `EscoRelationType` and `EscoRelation`;
- `EscoReference`;
- rebuildable `EscoOccupationSkill` projection;
- rebuildable `EscoHierarchyClosure` projection;
- indexes for graph traversal, localized lookup and projection access;
- append-only migration `20261006113000_esco_knowledge_graph_foundation`.

The schema preserves the original ESCO relation key/direction and keeps Higa semantic grouping separate from matching policy.

## Explicitly not included

This block does not include:
- normalized ESCO 1.2.0 snapshot import;
- dataset activation importer;
- projection population;
- runtime ESCO API/search;
- Candidate/Vacancy integration;
- matching weights or scoring;
- AI integration.

## Local verification

The project owner reported:

```text
npx prisma migrate dev
→ already in sync

npm run db:generate
→ Prisma Client 7.10.0 generated

npm run verify
→ Prisma schema valid
→ TypeScript typecheck passed
→ 19 test files passed
→ 92 tests passed
→ production build passed

npm run db:status
→ 10 migrations found
→ Database schema is up to date

git status
→ working tree clean
```

## Reopen conditions

Reopen this foundation only if:
- the importer exposes a schema flaw;
- source relation classification cannot be represented correctly;
- a new ESCO dataset version requires a change to stable concept/version semantics;
- measured query behavior demonstrates an index/model defect.

Normal importer/API/matching implementation belongs to subsequent blocks and does not reopen this closure by itself.
