# ESCO Knowledge Graph

Status: **RUNTIME SEARCH VERIFIED — SEMANTIC FILTERING CLOSED**
Last reviewed: 2026-10-06

This document records the backend persistence design for the HigaBase ESCO Knowledge Graph.

Cross-project invariants are owned by `RagueL-HigaBase/higabase_plans/ESCO_ARCHITECTURE.md`.

## 1. Scope

This backend block models ESCO as a versioned semantic graph in PostgreSQL.

Implemented in this backend:
- deterministic frozen ESCO 1.2.0 snapshot import and activation;
- canonical graph and rebuildable projections;
- multilingual runtime prefix search through `GET /api/esco/search`;
- independent search and display languages with English display fallback;
- optimized prefix access through `EscoSearchTerm`.

Not implemented here yet:
- Candidate/Vacancy persistence links;
- matching weights or percentages;
- AI/CV parsing.

The first dataset remains the frozen normalized ESCO 1.2.0 snapshot produced by `higa-esco-extractor`.

## 2. Verified source baseline

The source audit used to design this schema reports:

```text
concepts       19,070
occupations     3,665
skills         15,383
labels      1,040,825
texts         483,734
relations     404,098
references        542
status          CLEAN
```

Normalized relations have no dangling endpoints.

## 3. Persistence layers

The runtime model has three layers:

```text
STABLE IDENTITY
EscoConcept

VERSIONED SOURCE DATA
EscoDataset
EscoConceptVersion
EscoLabel
EscoText
EscoRelationType
EscoRelation
EscoReference

REBUILDABLE PERFORMANCE PROJECTIONS
EscoOccupationSkill
EscoHierarchyClosure
EscoSearchTerm
```

The projection tables are never canonical ESCO truth.

## 4. Stable concept identity

`EscoConcept` stores the official ESCO URI once.

Business domains will eventually reference the compact Higa concept id, while the URI remains the durable external identity.

Dataset-specific state belongs to `EscoConceptVersion`.

This allows a future ESCO dataset version to reuse stable concept identity without forcing Candidate/Vacancy foreign-key rewrites.

## 5. Concept kind and source family

Do not interpret `hb_kind` alone.

The audited source contains:
- rows with class/kind Occupation whose URI family is `isco`;
- rows with class/kind Skill whose URI family is `isced-f`.

Therefore runtime filters for ordinary ESCO occupations/skills must consider both kind and source family.

Examples:

```text
ordinary occupation:
kind = OCCUPATION
sourceFamily = occupation

ordinary skill:
kind = SKILL
sourceFamily = skill
```

ISCO and ISCED-F nodes remain graph/taxonomy concepts and must not be presented as ordinary profession/skill search results merely because their class maps to the same normalized kind.

## 6. Multilingual records

Labels and texts are version-scoped child records.

Labels preserve:
- normalized source identity;
- language;
- original source language key;
- preferred/alternative type;
- original value;
- normalized search value.

Texts preserve:
- normalized source identity;
- language/source language key;
- description/scope-note/definition type;
- original text;
- mimetype.

No language-specific columns are allowed.

## 7. Canonical relation graph

Every normalized source edge becomes an `EscoRelation`:

```text
dataset
sourceConcept
relationType
targetConcept
semanticGroup
sourceIdentity
```

`EscoRelationType.hb_key` stores the original ESCO relation key.

`hb_semantic_group` is Higa-derived traversal metadata and never replaces the original source relation key.

Semantic classification may depend on relation key plus endpoint kind/family.

## 8. Verified relation keys and classification rules

The snapshot contains 19 source relation keys.

Initial classification contract:

| Source relation | Endpoint pattern | Semantic group | Projection use |
| --- | --- | --- | --- |
| hasEssentialSkill | occupation → skill | OCCUPATION_SKILL | ESSENTIAL occupation-skill |
| isEssentialForOccupation | skill → occupation | OCCUPATION_SKILL | inverse validation |
| hasOptionalSkill | occupation → skill | OCCUPATION_SKILL | OPTIONAL occupation-skill |
| isOptionalForOccupation | skill → occupation | OCCUPATION_SKILL | inverse validation |
| hasEssentialSkill | skill → skill | SKILL_DEPENDENCY | graph only |
| isEssentialForSkill | skill → skill | SKILL_DEPENDENCY | graph only |
| hasOptionalSkill | skill → skill | SKILL_DEPENDENCY | graph only |
| isOptionalForSkill | skill → skill | SKILL_DEPENDENCY | graph only |
| broaderOccupation | occupation → occupation | OCCUPATION_HIERARCHY | hierarchy candidate |
| narrowerOccupation | occupation → occupation | OCCUPATION_HIERARCHY | hierarchy candidate |
| broaderSkill | skill → skill | SKILL_HIERARCHY | hierarchy candidate |
| narrowerSkill | skill → skill | SKILL_HIERARCHY | hierarchy candidate |
| broaderHierarchyConcept | skill → skill | SKILL_HIERARCHY | hierarchy candidate |
| broaderConcept / narrowerConcept | skill → skill | SKILL_HIERARCHY | hierarchy candidate |
| narrowerOccupation | isco → occupation | CLASSIFICATION | no occupation closure |
| broaderIscoGroup | occupation → isco | CLASSIFICATION | no occupation closure |
| narrowerSkill | isced-f → skill | CLASSIFICATION | no skill closure |
| broaderHierarchyConcept | skill → isced-f | CLASSIFICATION | no skill closure |
| broaderConcept / narrowerConcept | isco ↔ isco | TAXONOMY | taxonomy only |
| broaderConcept / narrowerConcept | isced-f ↔ isced-f | TAXONOMY | taxonomy only |
| broaderConcept / narrowerConcept | isced-f ↔ skill | CLASSIFICATION | no skill closure |
| isInScheme | concept → concept-scheme | TAXONOMY | no matching score |
| isTopConceptInScheme | concept → concept-scheme | TAXONOMY | no matching score |
| hasReuseLevel | skill → skill-reuse-level | REFERENCE_METADATA | metadata |
| hasSkillType | skill → skill-type | REFERENCE_METADATA | metadata |
| regulatedProfessionNote | occupation → regulated-professions | REFERENCE_METADATA | metadata |

Any endpoint pattern not explicitly classified remains `OTHER` until audited.

Do not infer a semantic group solely from the raw relation key.

## 9. Inverse-pair rule

ESCO contains inverse relation pairs.

Examples:

```text
occupation --hasEssentialSkill--> skill
skill --isEssentialForOccupation--> occupation

occupation --hasOptionalSkill--> skill
skill --isOptionalForOccupation--> occupation
```

The canonical graph preserves both directions when present.

The occupation-skill projection collapses the pair into one normalized fact.

Before projection population, the importer must validate inverse-pair consistency for the active dataset.

## 10. Occupation-skill projection

`EscoOccupationSkill` is optimized for fast capability-profile construction.

Fields:
- dataset;
- occupation concept;
- skill concept;
- importance: ESSENTIAL or OPTIONAL.

It is rebuildable from canonical relations.

The projection does not contain:
- Candidate state;
- Vacancy state;
- match score;
- overskill;
- custom employer requirements.

## 11. Hierarchy closure

`EscoHierarchyClosure` stores transitive hierarchy paths for high-volume traversal.

Fields:
- dataset;
- semantic group;
- ancestor concept;
- descendant concept;
- depth.

Only `OCCUPATION_HIERARCHY` and `SKILL_HIERARCHY` edges may feed the initial closure.

Classification/taxonomy/reference edges are excluded.

Before the closure builder is implemented, parent/child orientation for every accepted hierarchy source relation must be validated against real graph samples and inverse consistency.

## 12. Index strategy

The schema includes indexes for:
- dataset name/version and status;
- concept URI;
- dataset + concept kind;
- dataset + source family;
- label language/type;
- relation source + relation type;
- relation target + relation type;
- relation source/target + semantic group;
- direct source-target lookup;
- occupation → skill projection;
- reverse skill → occupation lookup;
- hierarchy ancestor/depth;
- hierarchy descendant/depth.

Runtime prefix search is backed by `hb_esco_search_terms_prefix_idx` on dataset + language + normalized value using PostgreSQL `text_pattern_ops`. This matches the prefix LIKE access pattern. Prisma 7.10 cannot represent this operator-class index in the schema DSL, so it is intentionally managed by the SQL migration `20261006133000_esco_search_prefix_index`.

Verified `operator` / Dutch / OCCUPATION benchmark on the active 1.2.0 dataset:
- before prefix index: SQL `115.331 ms`; 50,001 NL candidates read and 49,388 removed by the prefix filter; warm HTTP roughly `74–78 ms`;
- after prefix index: SQL `3.283 ms`; 613 prefix candidates reached directly; warm HTTP roughly `5–6 ms`.

The SQL path improved by about 35x and the observed warm HTTP path by about 13–14x. No Redis/cache layer is justified for this autocomplete path at the current measured performance. Fuzzy/trigram search remains deferred until a product requirement and benchmark justify it.

## 13. Dataset lifecycle

Dataset state:

```text
IMPORTING
ACTIVE
FAILED
ARCHIVED
```

The future importer must:
1. verify the normalized snapshot manifest/hash;
2. import canonical nodes and version records;
3. import labels/texts/references;
4. seed relation types;
5. import and classify all source relations;
6. verify counts and endpoints;
7. build occupation-skill projection;
8. build hierarchy closure;
9. audit projections;
10. atomically activate the dataset.

A failed import must not replace the current ACTIVE dataset.

## 14. Physical database names

All physical PostgreSQL objects use the `hb_` prefix.

Prisma models use the `Esco...` prefix for this reference domain.

Schema migration:

```text
20261006113000_esco_knowledge_graph_foundation
```

## 15. Verification status

The foundation schema/migration block was locally verified by the project owner on 2026-10-06.

Verified:
- `npx prisma migrate dev` — database already in sync;
- `npm run db:generate` — Prisma Client 7.10.0 generated;
- `npm run verify` — schema validation, typecheck, 19 test files / 92 tests, and production build passed;
- `npm run db:status` — 10 migrations found and database schema up to date;
- working tree clean.

The frozen ESCO 1.2.0 snapshot is imported and ACTIVE. Runtime search projection contains `1,040,825` terms and has passed importer audit.

Runtime semantic filtering was verified against the ACTIVE 1.2.0 dataset. OCCUPATION autocomplete requires source family `occupation` and excludes ISCO nodes. SKILL autocomplete requires source family `skill` and excludes ISCED-F nodes. Taxonomy/classification concepts remain preserved in the canonical graph and complete search projection.


## 16. Deterministic ESCO 1.2.0 importer

The backend now includes a maintenance importer:

```text
src/esco/import-cli.ts
src/esco/import-snapshot.ts
src/esco/relation-classifier.ts
src/esco/search-value.ts
```

Command:

```bash
npm run esco:import
```

Default snapshot location:

```text
../higa-esco-extractor/data/normalized/esco-1.2.0
```

Override when needed:

```bash
npm run esco:import -- --snapshot <path>
```

Optional batch-size override:

```bash
npm run esco:import -- --batch-size 5000
```

The importer:
1. validates every normalized data-file SHA-256 against `manifest.json`;
2. recomputes and validates the aggregate snapshot hash;
3. creates/reuses an `IMPORTING` dataset row;
4. imports stable concept identity and dataset-scoped concept versions;
5. streams labels/texts in batches;
6. seeds the 19 verified source relation keys;
7. imports every canonical relation with endpoint-aware semantic classification;
8. imports reference metadata;
9. validates Occupation↔Skill inverse pairs;
10. builds the occupation-skill projection;
11. builds hierarchy transitive closure;
12. compares canonical database counts with the manifest;
13. atomically archives a previous ACTIVE dataset and activates the new one only after all audits pass.

A failed import is marked `FAILED`. Re-running the same failed snapshot clears only that dataset's scoped rows and retries. A current ACTIVE dataset remains untouched until final activation.

`hb_search_value` is a derived NFKC/lowercase/whitespace-normalized label value. Diacritics are intentionally preserved; fuzzy/accent-folding behavior remains a later benchmark decision.

## 17. Historical importer verification procedure

The importer has subsequently been owner-verified on a real local ESCO 1.2.0 snapshot as recorded above and in `PROJECT_STATE.md`. The following commands are retained as a repeatable local re-verification procedure, not an open verification blocker.

Historical/reverification command sequence:

```bash
git pull
npm run typecheck
npm test
npm run esco:import
npm run verify
npm run db:status
```

The import can take materially longer than ordinary backend verification because it writes more than one million labels plus texts/relations and computes hierarchy projections.
