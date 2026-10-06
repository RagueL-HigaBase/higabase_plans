# HigaBase AI Core Architecture

Status: **FOUNDATION**
Last reviewed: **2026-10-06**

This document defines the cross-project AI boundary for HigaBase.

AI is an interpretation layer between unstructured or provider-specific information and strict Higa domain contracts. AI is not a domain authority, persistence owner, workflow owner, or substitute for deterministic business rules.

---

## 1. Core invariant

```text
UNSTRUCTURED / EXTERNAL INPUT
            ↓
        HIGA AI CORE
            ↓
   STRICT HIGA CONTRACT
            ↓
 VALIDATION / DOMAIN RULES
            ↓
 DRAFT / PROPOSAL / CHECKPOINT
            ↓
 CANONICAL DOMAIN PERSISTENCE
```

AI translates human/provider information into Higa's architecture.

AI must not invent a new persistence shape for each prompt or provider.

## 2. Provider-neutral boundary

Higa domain code must not depend directly on OpenAI, Anthropic, a local model, or another model provider.

Conceptually:

```text
Higa domain
    ↓
AI task contract
    ↓
Higa AI runtime
    ↓
provider adapter
    ├─ provider A
    ├─ provider B
    └─ future/local provider
```

Provider-specific request/response formats stop at the adapter boundary.

Switching a provider must not require redesigning Candidate, Resume, Vacancy, ESCO, workflow, or other domain persistence.

## 3. Task contracts

AI operations are explicit named tasks, not arbitrary free-form calls from domain code.

Examples:

```text
candidate.resume.parse
candidate.esco.propose
candidate.fact.compare
vacancy.parse
vacancy.esco.propose
message.classify
data.anomaly.explain
```

Each task defines:
- input contract;
- output contract;
- contract version;
- allowed capabilities;
- validation rules;
- failure behavior;
- provenance requirements.

The initial implementation should remain small and add tasks only when a real domain use case requires them.

## 4. AI output is untrusted

Every AI response is untrusted input until it passes the Higa task output schema and domain validation.

```text
model response
    ↓
parse
    ↓
strict schema validation
    ↓
semantic/domain validation
    ↓
accepted draft/proposal OR rejected result
```

Malformed, incomplete, structurally unexpected, or domain-invalid output must fail closed.

The system must not persist arbitrary model JSON into canonical domain tables.

Runtime schema validation should use the project's normal strict validation mechanism, such as Zod in the TypeScript backend.

## 5. No fabrication of facts

AI must distinguish source facts from inference.

If the source does not contain an employer, exact date, certificate, identifier, or other fact, AI must not fabricate one to complete the contract.

A result may explicitly represent:
- extracted;
- inferred;
- normalized;
- uncertain;
- conflicting;
- missing.

Unknown is a valid state.

Fabricated certainty is not.

## 6. Original source preservation

The original source artifact is immutable evidence and remains available in the Higa file/object storage layer.

Examples:
- original PDF CV;
- original Word CV;
- source image;
- source audio;
- imported source document.

AI/domain persistence should reference the source through a stable file/artifact reference rather than duplicating the binary document into AI records.

Conceptually:

```text
SourceArtifact
- stable artifact/file id
- storage reference
- content type
- checksum/version metadata
- created/imported metadata

AI execution / domain draft
- sourceArtifactId
- source location/reference when useful
- extraction/provenance metadata
```

Derived text may be stored when required for processing, search, audit, or reproducibility, but it never replaces the original artifact.

A later parser/model/version must be able to reprocess the original source.

## 7. Provenance

AI-derived information must retain enough provenance to understand how it was produced.

Useful provenance includes:
- source artifact/reference;
- source fragment/location when available;
- AI task name and contract version;
- provider/model execution metadata;
- prompt/instruction version or equivalent processing version;
- creation timestamp;
- confidence when meaningful;
- whether the value was extracted, inferred, or normalized;
- later human confirmation/override.

Provenance must support debugging, audit, reprocessing, comparison between processing versions, and human review.

## 8. Draft versus canonical truth

AI creates drafts, proposals, classifications, explanations, or comparisons.

AI does not silently create human-confirmed business truth.

For Candidate intake:

```text
original CV
    ↓
extraction
    ↓
AI Candidate/Resume draft
    ↓
strict validation
    ↓
domain normalization
    ↓
human checkpoint where required
    ↓
canonical Candidate/Resume
```

High-confidence deterministic results may reduce human work where domain policy permits, but the AI itself does not grant confirmation authority.

## 9. ESCO boundary

AI may interpret occupation/skill text and propose ESCO mappings.

AI must not invent an ESCO concept id or URI.

Valid mappings must resolve against Higa's active ESCO reference layer.

Conceptually:

```text
source occupation/skill text
        ↓
AI interpretation
        ↓
Higa ESCO candidate retrieval
        ↓
proposal restricted to real concepts
        ↓
validation
        ↓
confirmed/accepted mapping
```

Original Resume/Vacancy text remains preserved independently of the ESCO mapping.

## 10. Persistence boundary

AI Core does not own Candidate, Resume, Vacancy, ESCO, Organization, workflow, or component domain records.

It may own technical execution/provenance records where needed.

Canonical persistence is performed by the owning domain service after validation and authorization.

AI must never receive direct unrestricted Prisma/database write authority.

## 11. Workflow and authorization boundary

AI may:
- parse;
- classify;
- extract;
- normalize proposals;
- compare;
- summarize;
- explain;
- identify uncertainty;
- recommend a next action.

AI must not independently:
- grant access or permissions;
- approve Organization relationships;
- activate users or Organizations;
- publish externally;
- accept lifecycle checkpoints;
- override authoritative sources;
- silently resolve conflicts that require human/business authority.

Workflow and backend domain rules remain authoritative.

## 12. Failure behavior

AI failure must be explicit and contained.

The runtime must distinguish at least:
- provider unavailable/timeout;
- invalid provider response;
- output contract validation failure;
- domain validation failure;
- unsupported/insufficient source;
- uncertain result where task policy requires review.

Retries must be bounded and task-controlled.

A provider failure must not corrupt canonical domain state.

## 13. Reproducibility and versioning

AI behavior changes over time, therefore processing identity must be versioned.

At minimum Higa must be able to determine:
- which task contract produced a result;
- which processing/prompt version was used;
- relevant provider/model metadata;
- which source artifact/version was processed.

Changing a model or prompt must not silently rewrite historical confirmed facts.

Reprocessing creates a new derived result that can be compared/reviewed according to domain policy.

## 14. Data minimization

Only data required for the AI task should be sent to a provider.

Provider adapters must not automatically receive entire Candidate, Organization, or system records when the task requires only a subset.

Secrets, credentials, authorization tokens, password hashes, session data, and unrelated private data must never be included in AI task payloads.

Provider retention/privacy configuration is deployment/provider policy and must be reviewed before production use.

## 15. Observability

AI Core should expose technical metadata sufficient to monitor:
- task;
- success/failure;
- latency;
- provider/model;
- token/usage information when available;
- validation failures;
- retry count;
- processing version.

Observability data must not become an uncontrolled duplicate store of sensitive source content.

## 16. Candidate as first proving use case

Candidate CV intake is the first real use case that should prove AI Core.

Implementation sequence:

```text
AI-CORE foundation
→ Candidate Core
→ Resume Core
→ CV source artifact/upload
→ text/content extraction
→ candidate.resume.parse
→ strict Candidate/Resume draft validation
→ ESCO normalization proposal
→ recruiter review/checkpoint
→ canonical persistence
```

The architecture must be exercised against heterogeneous real CVs, not only one curated example.

## 17. Future reuse

The same AI boundary may later support:
- Candidate voice/text intake;
- Vacancy interpretation;
- Vacancy ESCO normalization;
- matching explanations;
- recruiter notes;
- messaging classification/routing;
- data consistency/anomaly explanation;
- external integration interpretation.

Reuse means common execution/validation/provenance infrastructure.

It does not mean one universal prompt or one universal output schema.

## 18. Non-goals of AI Core foundation

Not decided by this foundation:
- final AI provider;
- final model;
- final prompt wording;
- final database tables for execution logs;
- final file/object-storage provider;
- final retry thresholds;
- final confidence thresholds;
- final Candidate parser schema.

Those decisions must be driven by implementation evidence while preserving the boundaries above.

## 19. Mandatory summary

```text
SOURCE IS PRESERVED
AI INTERPRETS
HIGA CONTRACT CONSTRAINS
VALIDATION FAILS CLOSED
DOMAIN RULES DECIDE
HUMAN CONFIRMS WHERE REQUIRED
CANONICAL DATA REMAINS PROVIDER-INDEPENDENT
```
