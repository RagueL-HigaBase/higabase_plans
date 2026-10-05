# ESCO Knowledge Architecture

Status: **FOUNDATION**
Last reviewed: **2026-10-05**

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

Runtime storage preserves the semantic separation established by the normalized snapshot:

```text
EscoDataset
    ↓
EscoConcept
    ├─ EscoLabel[]
    ├─ EscoText[]
    └─ EscoRelation[]
```

`EscoConcept` is language-neutral identity. The official ESCO URI is the durable external identity.

Higa may use a compact internal identifier for joins/performance while preserving the official URI as a unique canonical reference.

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
- final Prisma schema;
- final HTTP endpoint naming;
- final database index implementation;
- dedicated search engine selection;
- final AI provider/model;
- final semantic matching algorithm;
- final cache strategy;
- final ranking weights.

These decisions come from implementation and benchmark evidence while preserving the boundaries above.
