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

Canonical `EscoLabel` remains the source of truth for multilingual terminology, but high-volume interactive lookup should use a rebuildable search projection rather than coupling search implementation to canonical storage.

```text
EscoLabel
    ↓ deterministic projection
EscoSearchTerm
    ↓ indexed lookup/ranking
Search result
```

Conceptual `EscoSearchTerm` fields:

```text
datasetId
conceptId
language
termType
value
normalizedValue
priority
```

The projection may contain preferred and alternative labels and may later contain other source-supported searchable terms. It must be fully rebuildable from canonical ESCO state and must never become the semantic source of truth.

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

The search projection and graph projections are approved rebuildable read models. External cache infrastructure and a dedicated search engine remain evidence-driven later optimizations.

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
EscoSearchTerm
EscoOccupationSkill
EscoHierarchyClosure
```

Canonical source tables:
- preserve the frozen snapshot semantics;
- retain URI identity;
- retain relation direction;
- remain dataset-aware.

Derived/read-model tables:
- `EscoSearchTerm`;
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

For `EscoSearchTerm`, the first implementation should support:
- language-scoped lookup;
- term-type/priority-aware ranking;
- normalized prefix lookup;
- efficient small top-N result retrieval.

PostgreSQL trigram indexing is the preferred first fuzzy-search option when benchmarked autocomplete requires typo/substring tolerance. A separate search engine remains deferred until PostgreSQL benchmarks demonstrate a real need.

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


## 27. Stable identity versus dataset version

External Higa domains should reference stable `EscoConcept` identity, not a localized label and not a row whose identity disappears when another dataset version is imported.

```text
EscoConcept
- id          compact Higa identity
- uri         official ESCO URI, UNIQUE

EscoConceptVersion
- datasetId
- conceptId
- kind
- sourceFamily
- classId?
- className
- code?
- status?
- source/version metadata
```

This separates two concerns:

```text
"What concept is this?"
→ EscoConcept

"What did ESCO dataset 1.2.0 say about this concept?"
→ EscoConceptVersion
```

Candidate/Vacancy mappings may additionally retain the dataset version used during normalization as provenance, but the durable semantic reference remains the stable concept.

A future dataset import must never silently remap an existing URI to a different Higa concept identity.

## 28. Internal identifier strategy

Runtime graph joins should use compact integer identifiers rather than official URI strings.

The URI remains canonical external identity, while internal IDs are implementation keys.

```text
external/canonical identity → ESCO URI
runtime relational identity → compact integer conceptId
```

This is especially important for relation traversal and derived projections, where the same concept identifiers participate in large numbers of joins.

Consumer APIs must not rely on the numeric value being globally portable outside the owning ESCO persistence boundary. The stable interoperability identifier is the ESCO URI.

## 29. Search normalization

Search normalization is a derived concern and must never modify canonical ESCO label values.

```text
EscoLabel.value
→ original ESCO value

EscoSearchTerm.normalizedValue
→ search-oriented normalized representation
```

Normalization may include operations proven safe for lookup, such as:
- Unicode normalization;
- case folding;
- whitespace normalization;
- punctuation normalization where appropriate.

Accent/diacritic-insensitive matching may be supported as a secondary search representation or database search operation when useful.

The original label is always preserved and returned for display.

Normalization rules must be deterministic and versioned with the projection/import logic so the search index can be rebuilt consistently.

## 30. Search-language and display-language separation

A search request has two language concerns:

```text
searchLanguage
→ language used to find candidate terms

displayLanguage
→ language used to present the resulting concept
```

For normal interactive UI search they are usually the same selected client language.

For AI/CV resolution they may differ.

Example:

```text
CV source term language = lv
recruiter UI language   = nl

resolve using lv
→ EscoConcept X
→ display preferred label in nl
```

Search results therefore conceptually contain:
- stable concept identity;
- matched term;
- matched language;
- preferred display label;
- display language;
- concept kind.

This allows diagnostics and AI provenance without forcing the UI to display the exact term that produced the match.

## 31. Language fallback policy

Language fallback must be explicit and ordered.

Interactive UI default:

```text
1. selected client language
2. English
3. stop
```

A broad multilingual search must not run automatically for every keystroke.

AI/source-document resolution may request broader language behavior when source-language detection is uncertain or when an extracted term is known to use a different language.

Fallback must not change concept identity. It changes only which label is used for search or presentation.

If no preferred display label exists in the requested language, the service may return a preferred English label while identifying the actual returned language.

## 32. Search ranking contract

Ranking belongs to the search/read layer, not to canonical ESCO.

The first ranking model should prefer deterministic lexical evidence before fuzzy evidence.

Conceptually:

```text
exact preferred
> prefix preferred
> exact alternative
> prefix alternative
> fuzzy preferred
> fuzzy alternative
```

Term-type priority and lexical quality should be represented independently enough that ranking can evolve without changing canonical labels.

The service should deduplicate results by concept so multiple matching labels do not flood autocomplete with the same semantic concept.

For each returned concept, the strongest matched term is retained as match evidence.

Final numeric weights remain benchmark-driven implementation details.

## 33. PostgreSQL search strategy

PostgreSQL is the initial search runtime.

The architecture does not require Elasticsearch/OpenSearch, a vector database or a separate search service for the current ESCO scale.

Recommended staged lookup:

```text
query
→ deterministic normalization
→ language + concept-kind scope
→ exact/prefix candidates
→ optional trigram fuzzy candidates
→ rank
→ deduplicate by concept
→ top N
→ resolve preferred display labels
```

Indexes should be designed around the actual access pattern rather than generic full-text indexing across all languages.

A universal PostgreSQL full-text configuration is not assumed for multilingual ESCO because language-specific linguistic processing differs and does not map uniformly across all supported ESCO languages.

Fuzzy/trigram behavior should be introduced and tuned from benchmark evidence.

## 34. Graph access strategy

Graph access and lexical search are separate runtime workloads.

Graph traversal operates on concept identifiers and relation semantics; it does not require labels except when results are finally presented.

```text
Search workload:
language + normalized term → concept

Graph workload:
concept + relation semantics → related concepts
```

Canonical relation indexes must support both directions:

```text
(datasetId, sourceConceptId, relationTypeId)
(datasetId, targetConceptId, relationTypeId)
```

A direct source-target lookup should also be efficient where validation or deduplication requires it.

High-frequency transitive hierarchy traversal should use the rebuildable hierarchy closure rather than repeatedly performing deep recursive traversal.

## 35. Consumer-owned mapping rule

A relationship between an Higa domain object and ESCO belongs to the consuming domain, never to ESCO.

Forbidden:

```text
EscoConcept
- candidateId
- vacancyId
```

Correct direction:

```text
Candidate domain
CandidateOccupationMapping
- candidateExperienceId
- sourceText
- sourceLanguage
- escoConceptId
- datasetVersion/provenance?
- mappingSource
- confidence?
- confirmation state?

Vacancy domain
VacancyOccupationMapping
- vacancy requirement/source fact
- sourceText
- sourceLanguage
- escoConceptId
- datasetVersion/provenance?
- mappingSource
- confidence?
- confirmation state?
```

Exact domain field names are not finalized here.

The invariant is:

```text
Consumer knows ESCO.
ESCO does not know Consumer.
```

Deleting or changing a Candidate/Vacancy mapping must never mutate canonical ESCO.

## 36. Protocol independence

ESCO Knowledge is a domain/service boundary, not a transport protocol.

Its conceptual operations remain stable whether invoked through:
- an in-process service;
- HTTP/REST;
- a future internal service protocol;
- an AI tool adapter.

```text
Consumer
   ↓
transport adapter
   ↓
EscoKnowledge contract
   ↓
canonical/read models
```

Do not design ESCO persistence around React, REST, GraphQL, gRPC or a specific AI provider.

Do not expose database tables directly as the public contract.

A future extraction into a separate deployable service must be possible without changing the semantic meaning of Candidate/Vacancy ESCO mappings.

## 37. Response-shape and payload discipline

Different use cases require different ESCO views.

Autocomplete should return a small result:

```text
concept identity
kind
display label
display language
matched term/language when useful
```

Concept detail may additionally request texts and metadata.

Graph/matching operations request relations/projections without automatically loading multilingual texts.

This avoids large object graphs and unnecessary multilingual payloads.

Frontend clients must not receive all labels/texts for a concept unless a specific editing/inspection use case requires them.

## 38. Cache boundary

Caching is an optimization above stable knowledge semantics.

Because an active ESCO dataset is effectively immutable during normal runtime, ESCO responses are highly cacheable.

Potential cache layers include:
- database buffer/cache behavior;
- application-local bounded cache for hot lookups;
- HTTP/client caching for stable concept detail;
- external distributed cache only when deployment evidence requires it.

Cache keys must include all dimensions that can change the result, especially:
- active dataset/version;
- operation;
- query/concept;
- search language;
- display language;
- concept kind/filter.

Cache must never become the source of truth.

## 39. Frontend performance contract

Web and Native clients consume ESCO incrementally.

They must not preload the complete ESCO label corpus.

Autocomplete clients should:
- wait for a small minimum useful query;
- debounce input;
- cancel/ignore stale requests;
- request a small result limit;
- cache recent query results where appropriate;
- persist selected semantic identity rather than the entire search response.

Exact timings and cache-library choices are client implementation decisions.

Changing the selected UI language invalidates language-dependent presentation/search cache entries but does not invalidate selected concept identity.

## 40. AI resolution contract

AI operates against the same ESCO Knowledge layer as deterministic UI search.

AI workflow:

```text
source text
→ detect document/term language
→ extract source occupation/skill term
→ query ESCO Knowledge
→ receive real candidate concepts
→ optional model disambiguation
→ validate selected concept
→ create consumer-owned proposal/mapping
```

The model must never synthesize an ESCO URI or concept ID that was not returned/validated by ESCO Knowledge.

AI resolution should preserve evidence:
- original source term;
- detected source language;
- matched ESCO term/language;
- selected concept;
- confidence/ranking information where useful.

## 41. Rebuildability invariant

All performance-oriented ESCO projections must be disposable.

```text
Canonical ESCO state
        ↓
   deterministic build
        ↓
SearchTerm / OccupationSkill / HierarchyClosure
```

If a derived table is lost or its algorithm changes, Higa must be able to rebuild it without consulting Candidate, Vacancy or external business data.

This rule prevents performance optimizations from becoming hidden sources of semantic truth.

## 42. Initial benchmark gates

Before adding infrastructure such as a dedicated search engine, distributed cache or vector database, benchmark the actual PostgreSQL implementation against representative workloads.

At minimum benchmark:
- occupation autocomplete by language;
- skill autocomplete by language;
- prefix and typo/fuzzy lookup where enabled;
- preferred-label resolution;
- outgoing/incoming direct relation traversal;
- occupation→skills projection;
- hierarchy closure lookup;
- concurrent Web/Native autocomplete patterns;
- AI batch resolution patterns.

Performance targets should be set from the deployment environment and UX requirements rather than invented in central architecture.

Architecture escalation happens only when measured evidence identifies a bottleneck.

## 43. Final separation of responsibilities

The resulting foundation is:

```text
                    FROZEN ESCO 1.2.0
                           │
                           ↓
                    CANONICAL CORE
       ┌───────────────────┼───────────────────┐
       ↓                   ↓                   ↓
 EscoConcept/Version    EscoLabel/Text     EscoRelation
       │                   │                   │
       │                   ↓                   ↓
       │             EscoSearchTerm      Graph projections
       │                   │                   │
       └──────────────┬────┴─────────────┬─────┘
                      ↓                  ↓
                EscoKnowledge        Matching
                      │
          ┌───────────┼────────────┐
          ↓           ↓            ↓
        Web         Native         AI
          │           │            │
          └────── semantic concept ┘
                      ↓
             consumer-owned mapping
          ┌───────────┴────────────┐
          ↓                        ↓
      Candidate                  Vacancy
```

Core invariants:
- ESCO is independent;
- concept identity is language-neutral;
- canonical values are preserved;
- search/read projections are rebuildable;
- graph and lexical search are separate workloads;
- consumers own mappings to ESCO;
- transport protocols do not shape persistence;
- optimization is benchmark-driven;
- ESCO 1.2.0 remains the frozen baseline for this implementation phase.

## 44. Approved physical Prisma/PostgreSQL schema contract

The first physical implementation follows the conceptual model above.

Canonical identity/version tables:

```text
EscoDataset
EscoConcept
EscoConceptVersion
EscoLabel
EscoText
EscoRelationType
EscoRelation
EscoReference
```

Rebuildable performance projections:

```text
EscoSearchTerm
EscoOccupationSkill
EscoHierarchyClosure
```

### EscoDataset

Owns one imported normalized snapshot and its lifecycle.

Important fields:
- integer runtime ID;
- dataset name;
- dataset version;
- extractor snapshot format version;
- SHA-256 snapshot hash;
- complete source manifest JSON;
- lifecycle status;
- import completion timestamp.

The snapshot hash is unique. Dataset/version lookup and status lookup are indexed.

### EscoConcept

Represents stable URI-backed identity across dataset versions.

Important fields:
- compact integer runtime ID;
- official ESCO URI, globally unique in Higa ESCO storage.

No language, Candidate, Vacancy or dataset-specific semantic fields belong here.

### EscoConceptVersion

Represents one concept as observed in one imported dataset.

It contains:
- dataset ID;
- stable concept ID;
- concept kind;
- source family;
- class ID/name;
- code/codes;
- source status;
- reference languages;
- source roles;
- reference families.

There is exactly one ConceptVersion per dataset + stable concept.

### EscoLabel

Canonical multilingual source label.

It preserves:
- concept-version ownership;
- extractor source identity;
- normalized language code when available;
- original source language key;
- preferred/alternative type;
- original source value.

It intentionally does **not** contain a search-normalized value. Search normalization belongs to `EscoSearchTerm`.

### EscoText

Canonical multilingual descriptive text.

It preserves extractor source identity, language/source language key, text type, original value and mimetype.

### EscoRelationType

Stores the source relation key independently from individual graph edges.

Inverse-key metadata may be recorded when verified. It must not synthesize source edges.

### EscoRelation

Canonical dataset graph edge.

It contains:
- dataset;
- deterministic extractor source identity;
- source concept;
- source relation type;
- target concept;
- Higa semantic group classification.

Uniqueness protects both source identity and duplicate source/type/target edges within a dataset.

Indexes support outgoing, incoming, semantic-group and direct source-target lookup.

### EscoReference

Preserves extractor reference-family membership and overlap metadata.

### EscoSearchTerm

Disposable lexical read model derived from canonical labels.

It contains:
- dataset;
- concept version;
- stable concept;
- source label;
- language;
- search term type;
- original searchable value;
- deterministic normalized value;
- ranking priority.

The source-label foreign key makes projection provenance explicit.

The initial relational indexes support language/type/priority scoping and concept/language resolution.

Prefix/fuzzy-specific PostgreSQL indexes are deliberately not frozen into the generic Prisma model. If benchmarks justify trigram lookup, the required PostgreSQL extension/index is introduced through an explicit SQL migration and documented as database-specific runtime infrastructure.

### EscoOccupationSkill

Disposable graph projection for verified occupation↔skill edges.

It stores dataset, occupation concept, skill concept and ESSENTIAL/OPTIONAL importance.

### EscoHierarchyClosure

Disposable transitive hierarchy projection.

It stores dataset, semantic hierarchy group, ancestor concept, descendant concept and graph depth.

Only verified hierarchy semantic groups may populate this table.

## 45. Physical ownership and deletion rules

Canonical stable identity is conservative:

```text
EscoConcept
→ never cascade-delete because a dataset/projection changes
```

Canonical dataset content is protected from accidental runtime deletion.

Projection rows are disposable and may cascade with their owning dataset/version/source label where appropriate.

Consumer-domain mappings are outside this schema and are not cascade children of ESCO dataset rows.

Dataset archival is the normal lifecycle operation after activation of a replacement dataset. Physical deletion is an explicit maintenance operation, not normal runtime behavior.

## 46. Search projection provenance invariant

Every first-generation `EscoSearchTerm` is traceable to a canonical `EscoLabel`.

```text
EscoSearchTerm
    ↓ sourceLabelId
EscoLabel
    ↓
EscoConceptVersion
    ↓
EscoConcept
```

This prevents an optimization layer from silently creating semantic terminology with no canonical source.

If future search enrichment introduces generated synonyms, transliterations or other non-source terms, they must be explicitly typed/provenanced rather than pretending to be canonical ESCO labels.

## 47. Prisma versus PostgreSQL-specific optimization

Prisma defines the portable relational structure, relations, enums, uniqueness and ordinary B-tree indexes.

PostgreSQL-specific performance features may require explicit SQL migrations.

Examples include:
- `pg_trgm` extension;
- GIN/GiST trigram indexes;
- expression indexes over normalized search representations;
- partial indexes if benchmarks identify a stable hot path.

These optimizations must remain compatible with the canonical/read-model separation and must be reproducible from migrations.

Do not add a PostgreSQL-specific index merely because it is theoretically useful. Add it after representative `EXPLAIN (ANALYZE, BUFFERS)` and application-level latency benchmarks demonstrate the access pattern.

## 48. Current implementation checkpoint

The existing `higa_systems_express` ESCO graph foundation is the current physical implementation target.

The approved correction after the search/read-model architecture review is:

```text
BEFORE
EscoLabel
- canonical value
- search-normalized value

AFTER
EscoLabel
- canonical value only

EscoSearchTerm
- source label provenance
- searchable value
- normalized value
- language/type/priority
```

This preserves the already-designed graph foundation while removing search-engine concerns from canonical label storage.

No Candidate or Vacancy mapping tables are added to the ESCO schema.
