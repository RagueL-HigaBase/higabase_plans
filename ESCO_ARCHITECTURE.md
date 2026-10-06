# ESCO Knowledge Architecture

Status: **FOUNDATION**
Last reviewed: **2026-10-06**

This document defines the cross-project ESCO knowledge boundary for HigaBase.

It is conceptual. The extractor, backend services, web client, mobile client, Candidate, Vacancy, AI and matching implementations may refine technical details but must not contradict these boundaries.

## 1. ESCO is an independent reference domain

ESCO is a shared semantic/reference domain. It is not owned by Candidate, Vacancy, AI, Matching or any client application.

```text
ESCO source
    ↓
higa-esco-extractor
    ↓
validated normalized snapshot
    ↓
Higa ESCO Knowledge
    ├─ Candidate
    ├─ Vacancy
    ├─ AI
    ├─ Matching
    ├─ Web
    └─ Native
```

Dependencies point toward ESCO Knowledge. ESCO Knowledge must not depend on Candidate or Vacancy domain state.

## 2. Frozen source baseline

The current HigaBase baseline is **ESCO 1.2.0**.

The existing validated normalized snapshot remains the source for the first Higa ESCO integration. Do not upgrade ESCO as part of the first knowledge-layer implementation.

Dataset version must nevertheless remain explicit so a future version can be imported without redefining consumer domains.

## 3. Extractor boundary

`higa-esco-extractor` remains a separate data-production tool.

Its responsibilities are source acquisition/preservation, normalization, multilingual labels/texts, relations, integrity audit and deterministic validated snapshot production.

The extractor is not the runtime ESCO API and must not become part of normal frontend request handling.

```text
source ESCO
→ extractor
→ frozen normalized snapshot
→ Higa importer
→ runtime ESCO Knowledge
```

## 4. Runtime knowledge model

Runtime storage preserves the semantic separation established by the normalized snapshot while separating stable concept identity from dataset-version state:

```text
EscoDataset
    │
    ├─ EscoConceptVersion
    │    ├─ EscoLabel[]
    │    └─ EscoText[]
    │
    ├─ EscoRelation[]
    ├─ EscoReference[]
    ├─ EscoOccupationSkill[]   (derived)
    └─ EscoHierarchyClosure[]  (derived)

EscoConcept
    └─ stable URI-backed identity across dataset versions
```

`EscoConcept` is language-neutral global identity. The official ESCO URI is the durable external identity.

Dataset-specific kind, family, class/code/status and multilingual values belong to `EscoConceptVersion` and its child records.

Higa uses compact integer identifiers for joins/performance while preserving the official URI as the unique canonical external reference.

Labels, texts and relations are separate records. Do not model multilingual data as columns such as `nameEn`, `nameNl` or `nameLv`.

## 5. Multilingual labels

A single ESCO concept may have labels in many languages. Concept identity does not change when display/search language changes.

```text
EscoConcept X
├─ nl → preferred/alternative labels
├─ en → preferred/alternative labels
├─ lv → preferred/alternative labels
├─ uk → preferred/alternative labels
└─ ...
```

Candidate and Vacancy reference the concept, not a copied localized ESCO name. Localized labels are resolved at presentation/search time.

Original source text from Resume or Vacancy input remains preserved independently.

## 6. System-language search and display

Interactive Higa UI search uses the selected system/client language as its primary ESCO language.

For business web users this normally derives from `SystemLanguage`. For candidate/mobile users it derives from candidate/mobile language context.

```text
selected client language
        ↓
ESCO search language
        ↓
localized autocomplete results
```

The frontend should not need to send a language parameter when authenticated/session context already provides the authoritative selected language.

Controlled fallback:

```text
1. selected system/client language
2. English fallback
3. optional broader multilingual resolution when explicitly required
```

Normal UI autocomplete should not mix unrelated languages into the first result set.

## 7. CV and source-document language is separate

UI language and source-document language are independent.

CV/Resume interpretation must not use the recruiter's Higa interface language as the semantic source language.

```text
UI search/display
→ selected SystemLanguage/client language

CV parsing
→ detected document/source language

ESCO identity
→ language-neutral EscoConcept
```

The CV pipeline detects document language before semantic normalization.

Mixed-language documents must be supported. A document may have one dominant language while individual occupation/skill terms use another language. Individual extracted terms may carry their own detected language for ESCO resolution.

After resolution, the same concept can be displayed in the current user's selected Higa language.

## 8. Search architecture

The runtime ESCO layer is read-heavy and optimized for lookup/search rather than mechanically mirroring extractor files.

The frontend must never download the full ESCO dataset or large multilingual label sets for ordinary autocomplete.

```text
Web / Native
    ↓ small search request
ESCO Knowledge Service
    ↓ indexed label lookup
ESCO storage
    ↓
small localized result DTO
```

Search primarily operates on indexed `EscoLabel` records.

Typical behavior: small minimum input length, client debounce, small result limit, language-scoped search, preferred display label with concept identity, and no graph/large relation payload in autocomplete.

Exact debounce/minimum-length/result-limit values are implementation benchmark decisions.

## 9. Search labels versus display labels

Searchable terminology and visible terminology are separate concerns.

Preferred/alternative and other source-supported terms may resolve a concept; UI normally displays the preferred localized label.

If future extractor work exposes terms intended for search/text mining but not presentation, they remain searchable without becoming normal visible labels.

```text
searchable term
≠ necessarily displayable term
```

## 10. Knowledge service boundary

Consumers use a stable ESCO Knowledge boundary rather than extractor files directly.

Conceptual capabilities:

```text
searchOccupations(query, language)
searchSkills(query, language)
resolveTerm(term, language, kind)
getConcept(conceptId)
getLocalizedLabel(conceptId, language)
getRelations(conceptId, relationType?)
```

These are capabilities, not final endpoint names.

Web/Native use localized autocomplete; Candidate/Vacancy use stable references; AI uses multilingual resolution; Matching uses concepts and relations.

Do not create independent ESCO datasets/normalization logic per consumer.

## 11. Performance principles

Initial runtime architecture favors one indexed relational knowledge store unless benchmarks demonstrate a real need for additional search infrastructure.

Runtime optimization focuses particularly on label search and relation traversal.

Core rules:
- compact internal concept identifiers may be used for runtime joins;
- official URI remains unique and durable;
- index label language, type and normalized search representation;
- return small DTOs;
- do not return texts/relations unless required;
- paginate/limit discovery endpoints;
- extractor/raw snapshot access stays outside request-time paths;
- benchmark real UI and AI workloads before adding a separate search engine.

Cache, dedicated search engine or precomputed read models are evidence-driven later optimizations.

## 12. Candidate and Vacancy references

Candidate and Vacancy preserve their own source facts and reference ESCO additively.

```text
CandidateExperience
occupationText = "heftruckchauffeur"
        ↓
ESCO normalization
        ↓
escoConceptId = X
```

Source text remains Candidate-owned data. Localized ESCO label remains ESCO-owned reference data.

## 13. AI boundary

AI does not own ESCO truth.

AI may extract terms, identify source language, request ESCO candidates, rank/disambiguate candidates and propose mappings.

AI must not invent ESCO identifiers. ESCO Knowledge is authoritative for concept existence.

```text
source text
→ language-aware extraction
→ ESCO Knowledge lookup
→ candidate concepts
→ optional AI disambiguation
→ validated EscoConcept reference
```

## 14. Matching boundary

ESCO supplies semantic concepts/relations. Matching remains separate.

```text
Candidate ──┐
            ├─ Matching Engine ← ESCO Knowledge
Vacancy ────┘
```

ESCO must not contain Candidate/Vacancy matching state or scoring rules.

## 15. First implementation sequence

```text
1. Keep ESCO 1.2.0 snapshot frozen
2. Define runtime ESCO persistence/read model
3. Build deterministic snapshot importer
4. Verify imported counts/integrity against snapshot manifest
5. Implement language-aware Knowledge Service
6. Implement indexed occupation/skill autocomplete
7. Benchmark Web/Native search behavior
8. Expose stable ESCO resolution contract for Candidate/CV AI
9. Reuse the same knowledge layer for Vacancy and Matching
```

Importer must be repeatable and dataset-aware. Import validation fails closed on integrity/structural mismatch.

## 16. Current non-goals

Not part of this foundation:
- upgrading beyond ESCO 1.2.0;
- final HTTP endpoint naming;
- dedicated search engine selection;
- final AI provider/model;
- final semantic matching algorithm;
- final cache strategy;
- final ranking weights;
- Candidate/Vacancy persistence integration;
- importer implementation and projection population.

The PostgreSQL/Prisma runtime graph foundation and its core indexes are now an approved architecture decision. Search-engine-specific indexes and matching weights remain evidence-driven later work.

## 17. Verified ESCO 1.2.0 graph baseline

The frozen normalized snapshot used for the first runtime integration is verified CLEAN and contains:

```text
concepts       19,070
occupations     3,665
skills         15,383
labels      1,040,825
texts         483,734
relations     404,098
references        542
```

There are no dangling normalized relation endpoints.

The graph is therefore sufficiently complete for runtime graph modeling without inventing missing nodes.

## 18. Source graph versus semantic interpretation

The source graph must be preserved losslessly.

```text
EscoConcept
    ↑
EscoRelation
    ↓
EscoConcept
```

Every source relation retains:
- dataset;
- source concept;
- original relation key;
- target concept;
- deterministic source identity.

Higa may additionally classify a relation into a semantic group for runtime traversal.

The semantic group is derived metadata. It must never replace the original ESCO relation key.

A raw relation key cannot always be assigned one semantic meaning in isolation. Classification may depend on:
- relation key;
- source concept kind;
- source URI family;
- target concept kind;
- target URI family.

Example: `broaderHierarchyConcept` occurs both Skill→Skill and Skill→ISCED-F. The first may participate in skill hierarchy traversal while the second is taxonomy/classification context.

## 19. Verified relation families

The current ESCO 1.2.0 snapshot exposes the following source relation keys:

```text
isEssentialForOccupation
hasEssentialSkill
hasOptionalSkill
isOptionalForOccupation
isInScheme
narrowerSkill
hasReuseLevel
hasSkillType
broaderHierarchyConcept
isTopConceptInScheme
broaderSkill
isOptionalForSkill
regulatedProfessionNote
narrowerOccupation
broaderIscoGroup
broaderOccupation
broaderConcept
narrowerConcept
isEssentialForSkill
```

Important inverse pairs include:

```text
occupation --hasEssentialSkill--> skill
skill      --isEssentialForOccupation--> occupation

occupation --hasOptionalSkill--> skill
skill      --isOptionalForOccupation--> occupation

occupation --broaderOccupation--> occupation
occupation --narrowerOccupation--> occupation

skill --hasOptionalSkill--> skill
skill --isOptionalForSkill--> skill

skill --hasEssentialSkill--> skill
skill --isEssentialForSkill--> skill

concept --broaderConcept--> concept
concept --narrowerConcept--> concept
```

Both directions remain in the canonical source graph when ESCO provides them.

Derived projections may collapse inverse pairs into one normalized runtime fact.

## 20. Semantic relation groups

Runtime relations may be classified into these Higa semantic groups:

```text
OCCUPATION_SKILL
OCCUPATION_HIERARCHY
SKILL_HIERARCHY
SKILL_DEPENDENCY
TAXONOMY
CLASSIFICATION
REFERENCE_METADATA
OTHER
```

These groups are for traversal/query semantics.

They are not matching percentages and must not contain Candidate/Vacancy business policy.

## 21. Occupation-skill projection

The source graph contains a large verified Occupation↔Skill layer.

For runtime use, build a rebuildable projection:

```text
EscoOccupationSkill
- dataset
- occupationConcept
- skillConcept
- importance = ESSENTIAL | OPTIONAL
```

The projection is derived from canonical relations such as the verified essential/optional Occupation↔Skill inverse pairs.

It must not become a second source of truth.

Its purpose is fast construction of an occupation capability profile:

```text
Occupation
→ essential skills
→ optional skills
```

Matching policy remains outside this projection.

## 22. Hierarchy closure projection

Hierarchical traversal should not require repeated recursive graph walking for every high-volume matching request.

A rebuildable transitive closure may store:

```text
EscoHierarchyClosure
- dataset
- semanticGroup
- ancestorConcept
- descendantConcept
- depth
```

Only verified hierarchy semantics participate in the closure.

Taxonomy/classification edges must not be silently mixed into skill/occupation hierarchy closure.

The stored depth represents graph distance within the selected hierarchy semantic group.

## 23. Matching-oriented interpretation boundary

Higa matching later compares semantic profiles, not occupation labels.

Foundation principle:

```text
Occupation gives context.
Skills give evidence.
Relations give proximity.
Business rules give meaning.
```

Exact concept identity is the strongest evidence for an individual skill requirement.

Non-exact concepts may later contribute through verified graph relationships and distance.

The matching layer, not ESCO persistence, determines:
- partial contribution;
- missing requirements;
- transferable capability;
- relevant additional skills;
- overskill signals;
- final fit score.

An overall requirement-fit score must not exceed 100%.

Additional/overskill capability is reported separately rather than inflating requirement coverage above 100%.

## 24. PostgreSQL runtime foundation

The first backend runtime schema uses:

```text
EscoDataset
EscoConcept
EscoConceptVersion
EscoLabel
EscoText
EscoRelationType
EscoRelation
EscoReference
EscoOccupationSkill
EscoHierarchyClosure
```

Canonical source tables:
- preserve the frozen snapshot semantics;
- retain URI identity;
- retain relation direction;
- remain dataset-aware.

Derived tables:
- `EscoOccupationSkill`;
- `EscoHierarchyClosure`.

Derived tables must be safely rebuildable from canonical graph state.

## 25. Indexing principles

The initial relational indexes optimize:
- dataset/version lookup;
- concept URI identity;
- concept kind/family filtering;
- localized label lookup;
- outgoing graph traversal;
- incoming graph traversal;
- semantic-group traversal;
- direct source-target edge lookup;
- occupation→skill projection lookup;
- skill→occupation reverse lookup;
- ancestor/descendant closure traversal.

Dedicated fuzzy/trigram/search-engine indexes remain deferred until real search benchmarks justify them.

## 26. Dataset activation and import safety

A future importer must use dataset lifecycle state:

```text
IMPORTING
→ ACTIVE

IMPORTING
→ FAILED

previous ACTIVE
→ ARCHIVED only through an explicit activation transition
```

A new dataset must not become active before:
- manifest/hash validation;
- canonical row counts/integrity checks;
- relation endpoint verification;
- derived projection build;
- final database audit.

A failed import must not damage the currently active dataset.

