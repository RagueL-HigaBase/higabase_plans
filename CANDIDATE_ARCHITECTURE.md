# Candidate Architecture

Status: **FOUNDATION**
Last reviewed: **2026-10-08**

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
├─ stable identity
├─ Candidate-owned profile/contact data
└─ references from Resume and optional component domains
```

Candidate does not own Organization-specific operational data. Organization relationship, access and transfer policy are separate concerns and must not shape the Candidate core data standard.

Optional component data must reference Candidate by `candidateId`; Candidate must not grow component-specific columns.

## 2. Resume boundary

Resume is an independent professional/source dataset linked to Candidate.

Resume data remains Resume data. Candidate Profile may use appropriate confirmed facts, but Resume is not flattened into Candidate and Candidate does not become a copy of a CV.

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

### 2.1 Resume Draft research contract

Before freezing the production Resume schema, HigaBase uses a disposable `ResumeDraftV1` research contract.

Initial blocks:
- personal/header data present in the source CV;
- contacts present in the source CV;
- summary/about text;
- languages and stated levels;
- professional skills;
- software/tools proficiency;
- work history;
- education;
- certifications;
- achievements;
- additional information.

The draft contract is intentionally allowed to change while heterogeneous CV samples are tested. It must not be treated as the final Candidate or Resume persistence model.

First experimental vertical slice:

```text
original CV
→ content extraction
→ candidate.resume.parse
→ strict ResumeDraftV1 validation
→ draft persistence
→ development/test JSON route
```

ESCO normalization, Organization permissions and production Candidate persistence are outside this first experiment.

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

## 14. Candidate basic profile v1

The first production Candidate implementation must start from a deliberately small profile.

The goal is not to model the entire CV. The goal is to create the minimum stable Candidate core that can support intake, confirmation and later Resume/ESCO enrichment.

Initial basic profile scope:

```text
Candidate
├─ basic identity
│  ├─ first name
│  ├─ middle name? 
│  └─ last name
├─ contact
│  ├─ email?
│  └─ phone?
└─ languages
   ├─ language
   └─ stated level?
```

Additional fields may be added only when a real Candidate flow requires them.

The first implementation must not depend on final Experience, Occupation, Skill, matching, seniority or component schemas.

### 14.1 Verification is state, not duplicate storage

Do not create separate "verified profile" and "unverified profile" copies.

The system keeps one canonical Candidate profile and separates:
- source/draft information;
- human confirmation state;
- screening workflow/history.

Conceptually:

```text
CV / input
   ↓
ResumeDraft / extracted source facts
   ↓
Candidate basic profile
   ↓
screening / confirmation
   ↓
same Candidate profile becomes screened
```

A verification session may be temporary/history data, but it must not become a second Candidate profile.

For the first implementation, keep Candidate profile screening state minimal:

```text
UNSCREENED
SCREENED
```

"In progress" belongs to the active screening/request workflow rather than requiring another permanent copy of the profile.

## 15. Candidate may relate to many Organizations

Candidate is global Candidate-domain identity.

One Candidate may have active relationships with multiple Organizations.

Conceptually:

```text
Candidate
├─ relation → Organization A
├─ relation → Organization B
└─ relation → Organization C
```

The Candidate profile is not duplicated per Organization.

Organization-specific access/relationship state is separate from global Candidate profile/screening state.

A screened Candidate remains screened when another Organization requests a relationship, unless the Candidate profile itself later requires re-screening under an explicit future policy.

## 16. CV intake with hidden identity resolution

The recruiter-facing flow must remain simple and must not reveal whether the Candidate already exists in Higa.

Recruiter action:

```text
Upload CV
→ Add candidate
→ system processes
→ request sent
```

Behind the interface, the system may:
1. normalize available email/phone;
2. resolve whether they belong to an existing Candidate;
3. create a new Candidate when no safe match exists;
4. reuse an existing Candidate when identity resolution is safe;
5. create a temporary Organization-relation request;
6. choose the correct Candidate confirmation/screening flow;
7. send the private email/link.

The recruiter must not be asked to choose "existing vs new Candidate".

The recruiter must not receive an account-existence signal through different success responses.

Identity conflicts or ambiguous email/phone matches are backend exceptions and must not be exposed as an existence lookup surface.

## 17. Candidate relation request and consent

A Candidate-Organization relationship becomes active only after Candidate confirmation/consent.

Before that, use a temporary request/workflow state rather than treating the Candidate as an active Organization relation.

Conceptually:

```text
Organization uploads CV
        ↓
identity resolution
        ↓
temporary relation request
        ↓
Candidate receives private link
        ↓
Candidate confirms
        ↓
screening required?
   ┌────┴────┐
   │         │
  no        yes
   │         ↓
   │     quick screening
   │         │
   └────┬────┘
        ↓
active Candidate ↔ Organization relation
```

If the Candidate already has a screened profile:
- the Candidate receives a relationship-confirmation request;
- no duplicate general screening is required;
- confirmation can activate the Organization relation immediately.

If the Candidate is not screened:
- confirmation continues into the same simple screening flow;
- the relation becomes ready only after the required screening is complete.

Future Organization-specific questions, if introduced, are separate from the global Candidate screening state.

## 18. Recruiter-facing derived states

The frontend must present simple business states, not backend implementation states.

Initial recruiter-facing Candidate intake states:

```text
Processing
Waiting for candidate
Needs screening
Ready
```

Meaning:

- `Processing` — CV/input is being parsed, normalized or identity-resolved.
- `Waiting for candidate` — private confirmation request has been sent and Candidate action is pending.
- `Needs screening` — general screening is still required and may be completed by the Candidate or recruiter.
- `Ready` — Candidate relationship is active and the Candidate profile is screened/usable.

These are presentation/read-model states. They do not require four matching database tables or four permanent domain entities.

The backend may use more detailed internal states, but the normal recruiter UI should expose only the next useful action.

## 19. Minimal interaction rule

Candidate intake must be optimized around simple actions.

Recruiter:

```text
UPLOAD → DONE
```

Candidate:

```text
CONFIRM → DONE
```

or, when required:

```text
CONFIRM → SCREEN → DONE
```

Do not introduce extra dialogs or steps merely to expose backend mechanics.

The architectural rule is:

```text
COMPLEXITY LIVES UNDER THE HOOD
THE FRONTEND SHOWS THE NEXT ACTION
```

This rule applies to both recruiter-facing Higa Systems and Candidate-facing clients.



## 20. Candidate Core storage convention — approved foundation (2026-10-08)

The owner approves independent, narrowly scoped physical tables in the shared Higa PostgreSQL convention. **Physical names use lowercase `hb_` (not mixed-case `HB_`)** to match existing database objects and migrations. Prisma models use `Candidate...` to distinguish this domain from `System...` and `Esco...`.

Initial persistence:
- `Candidate` → `hb_candidates`: identity `id`, `firstName`, `lastName`, timestamps; no organization owner or contacts in the same table.
- `CandidateContact` → `hb_candidate_contacts`: separate 0..1 contact record per candidate with optional email/phone and timestamps. Neither email nor phone is globally unique; identity resolution is a separate protected workflow.

Future independent tables (names are proposals until individual contracts are approved):
- Candidate language, address, resume document/source, work experience, education, skills, certification, verification/screening;
- Candidate–Organization relationship request/consent and access/visibility;
- append-only Candidate audit/history with actor type, actor reference and historical identity snapshot as appropriate.

Core tables do **not** include direct `organizationId`, `createdByUserId`, a JSON all-purpose profile, placeholder permissions or inferred ESCO capabilities. Creation actor and organization context must be captured in the later **transactional intake/audit flow**, before production create APIs become available.

Indexes must serve actual queries: UUID primary key; unique candidate FK on 1:1 contact; avoid redundant indexes, broad speculative indexes and unbounded JSON blobs. Candidate list access will use an indexed organization relation and keyset pagination once its flow is approved; do not expose a global unscoped candidate list.

**Gate:** Initial schema-only migration is not a live Candidate intake. No Candidate write/read API or seeded personal data may be enabled until minimal authorization, consent/request boundary and transactional audit are implemented. Global person names and CV facts must allow Unicode, unlike the existing business SystemUser ASCII-only rule.

The owner has explicitly approved preparing this minimal, separate Candidate storage foundation for consumption by both web Recruitment and future React Native. Backend repository scope must record an isolated exception; this does not transfer candidate identity ownership to business SystemUser and does not approve future mobile auth flows.
