# Candidate Architecture

Status: **FOUNDATION**
Last reviewed: **2026-10-05**

This document expands the Candidate-specific architecture referenced by `SYSTEM_ARCHITECTURE.md`.

It is conceptual and cross-project. Backend, web and mobile implementations may refine technical details but must not contradict these boundaries.

## 1. Candidate domain

Candidate is a first-class HigaBase domain object.

Candidate is not:
- a business `SystemUser`;
- an Organization component;
- a temporary parser result;
- an ESCO concept container.

Candidate provides a stable identity for the professional/person domain.

Conceptually:

```text
Candidate
├─ identity / contact
├─ Resume
├─ verification/provenance
└─ references from optional component domains
```

Optional component data must reference Candidate by `candidateId`; Candidate must not grow component-specific columns.

## 2. Resume boundary

Resume is a base Candidate layer.

It stores human/source professional facts such as:
- employment experience;
- education;
- languages;
- skills;
- certifications;
- other professional history.

Original source values must be preserved.

ESCO normalization is additive:

```text
Original Resume fact
        ↓
normalization proposal
        ↓
EscoConcept reference
```

Do not overwrite original Resume text with normalized labels.

## 3. Intake

The first production Candidate path should be CV-driven intake.

```text
Candidates
→ Add Candidate
→ Upload CV
→ Extract content
→ AI parse
→ Candidate/Resume draft
→ ESCO proposals
→ Review/correct
→ Save
→ Candidate dossier
```

The first intake implementation should be tested against a heterogeneous corpus of real CVs rather than one curated example.

Useful test coverage includes:
- simple one-page CVs;
- long multi-page CVs;
- PDF and Word;
- tables/columns;
- multiple languages;
- missing dates;
- overlapping employment;
- multiple occupations;
- inconsistent employer/title naming;
- incomplete contact data.

The goal is to discover architecture/data-contract gaps, not merely demonstrate successful AI output.

## 4. Draft versus confirmed state

AI parsing creates a draft.

A parsed value may be:
- extracted directly;
- inferred;
- normalized;
- uncertain;
- conflicting;
- missing.

The system should preserve provenance and confidence where useful.

Conceptual fact metadata:

```text
value
source
sourceReference?
confidence?
confirmedBy?
confirmedAt?
```

AI is not a human verification source.

## 5. Recruiter verification

After parsing, recruiter verification should minimize repeated manual entry.

The verification UI should derive questions from actual gaps and uncertainty rather than presenting a second full profile form.

Typical outcomes:

```text
YES
NO
UNKNOWN
EDIT
```

Examples:
- confirm employment dates;
- confirm current occupation;
- confirm certificates;
- confirm language level;
- fill missing availability;
- resolve conflicting values.

Verification sessions are workflow/history records.

They are not duplicate Candidate profile storage.

## 6. ESCO integration

ESCO is the semantic reference for professional concepts.

First real integration target:

```text
CandidateExperience
→ occupation text
→ normalization
→ EscoConcept
```

Then extend to skills and related concepts.

Mappings should retain:
- source Resume fact;
- selected EscoConcept;
- mapping source;
- confidence when AI-assisted;
- confirmation/override where applicable.

ESCO normalization should be reusable by matching and downstream components.

## 7. Candidate dossier

The dossier is a presentation/operation shell.

It consolidates:
- Candidate core;
- Resume;
- verification state;
- normalized semantic views;
- enabled component surfaces.

The dossier does not own all displayed data.

It resolves domain data from the owning layer.

## 8. Component model

Optional components are extensions around Candidate.

General rule:

```text
Component receives candidateId
→ stores its own Candidate-linked records
→ exposes its own operations/data
→ Candidate dossier may present the component surface
```

Examples:
- Housing;
- Vehicle;
- Planning;
- Hours;
- Planbition;
- future external integrations.

A new component must not require a Candidate schema migration merely to add component-specific state.

## 9. Component discovery layers

Keep separate:

```text
Organization capability
Candidate participation/data
User authorization
```

These answer different questions.

A dossier menu entry may later be derived from all three.

Some components may appear whenever enabled; others only when Candidate-linked data exists.

Do not force one universal visibility policy prematurely.

## 10. Candidates as core product surface

Candidates is a core business domain, not an App capability.

Client navigation may expose Candidates as a top-level business surface.

The exact visual/navigation placement is client-specific, but architecture must not treat Candidate management as an optional App merely because optional Apps can extend Candidate.

A future Candidates surface is expected to support:
- Candidate list/search;
- Add Candidate;
- CV intake;
- Candidate dossier navigation.

## 11. First implementation sequence

Recommended sequence:

```text
1. Candidate Core
2. Resume Core
3. Candidate Intake
4. ESCO normalization v1
5. Recruiter Verification
6. Component foundation
7. First real external component
```

Planbition is a suitable first external component candidate because it can force the generic component boundary through a real integration.

Do not build the generic component framework independently of a real component use case.

## 12. Permissions boundary

Final fine-grained permissions remain deferred.

Candidate routing and component surfaces should be structured so future authorization can attach to clear domain routes/actions.

Permissions should constrain an established architecture, not define Candidate/domain ownership.

## 13. Current non-goals

Not part of this foundation:
- final Candidate database schema;
- final AI provider/model selection;
- final parser implementation;
- final component registry schema;
- final permission matrix;
- final scoring thresholds;
- production ESCO matching algorithms.

These should be decided from implementation evidence while preserving the invariants above.
