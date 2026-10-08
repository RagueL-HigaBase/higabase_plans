<!-- Archived source from https://github.com/RagueL-HigaBase/higa_systems_express/blob/main/docs/production/ESCO_SNAPSHOT_IMPORTER.md; not current canonical state. -->

# ESCO 1.2.0 Snapshot Importer — Verified Closure

Status: **CLOSED**
Verified: **2026-10-06**

Canonical domain document:
- `docs/ESCO.md`.

## Closed scope

This block implemented and verified deterministic import of the frozen normalized ESCO 1.2.0 snapshot into the HigaBase PostgreSQL Knowledge Graph.

Implemented:
- normalized manifest/file SHA-256 validation;
- aggregate snapshot-hash validation;
- dataset lifecycle with `IMPORTING`, `FAILED`, `ACTIVE`, `ARCHIVED`;
- stable concept identity import;
- dataset-scoped concept versions;
- multilingual labels and texts;
- 19 verified source relation keys with endpoint-aware semantic classification;
- canonical relation import;
- reference import;
- Occupation↔Skill inverse-pair audit;
- `EscoOccupationSkill` projection generation;
- `EscoHierarchyClosure` generation with cycle/depth audits;
- canonical database-count comparison;
- fail-closed activation that preserves an existing ACTIVE dataset until final success;
- repeat invocation support for an already ACTIVE identical snapshot.

## Verified ACTIVE dataset

```text
datasetId          1
datasetVersion     1.2.0
status             ACTIVE
snapshotHash       6239309e7f3c9371d5f0d0d2208b8494d26e171292339c7b1da50fa04cc00077

concepts           19,070
labels          1,040,825
texts             483,734
relations         404,098
references            542
occupationSkills  129,004
hierarchyClosure   71,652
```

## Local verification

The project owner reported:

```text
npm run typecheck
→ passed

npm test
→ 21 test files passed
→ 99 tests passed

npm run esco:import
→ snapshot hashes valid
→ canonical import completed
→ inverse-pair audit passed
→ occupation-skill projection built
→ hierarchy closure built
→ final count audit passed
→ dataset=1 status=ACTIVE

npm run verify
→ Prisma schema valid
→ TypeScript passed
→ 21 test files / 99 tests passed
→ production build passed

npm run db:status
→ 11 migrations found
→ Database schema is up to date

git status
→ working tree clean
```

## Clean-database deployment verification

On 2026-10-06 the importer was additionally verified against a newly created empty PostgreSQL database. All 13 canonical migrations were applied from zero with `prisma migrate deploy`; the frozen ESCO 1.2.0 snapshot then imported to ACTIVE with the same snapshot hash and canonical/projection counts. Final database status was up to date, and backend verification passed 22/22 test files, 102/102 tests, TypeScript, Prisma validation, and production build.

This confirms that the ESCO runtime can be reproduced from repository migrations plus the frozen normalized snapshot without copying an existing development database.

## Explicitly not included

This block does not include:
- ESCO HTTP/search/read APIs;
- Candidate/Vacancy ESCO references;
- matching weights or scoring;
- AI integration;
- frontend/native ESCO interfaces;
- ESCO version upgrades.

## Reopen conditions

Reopen this importer block only if:
- the same frozen snapshot cannot be imported deterministically into a fresh compatible database;
- source/hash verification becomes inconsistent;
- projection/count audits diverge from the canonical snapshot;
- dataset activation can corrupt or replace an existing ACTIVE dataset prematurely;
- a schema defect prevents lossless import.

Normal Knowledge Service/search, Candidate/Vacancy integration and Matching work belong to subsequent blocks.
