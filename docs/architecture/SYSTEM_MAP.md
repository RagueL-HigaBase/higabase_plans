# HigaBase — System and Domain Map

Status: architecture navigation (not a replacement for canonical specifications). Updated 2026-10-08.

## Source authority

- [System architecture](../../SYSTEM_ARCHITECTURE.md) governs identity, boundaries, shared flows, permissions and data authority.
- [Candidate architecture](../../CANDIDATE_ARCHITECTURE.md) governs candidate identity, Resume, consent and verification.
- [ESCO architecture](../../ESCO_ARCHITECTURE.md) governs immutable reference knowledge, datasets, relations and multilingual search.
- [AI Core](../../AI_CORE.md) governs untrusted interpretation and provider adapters.
- [Project state](../../PROJECT_STATE.md) records implementation evidence and remaining blockers.

## Runtime boundaries

```text
Business User (SystemUser) --authenticated session--> Express Business API
                                                |
React/Phoenix business client ------------------+
                                                |
                                      System Organization
                                             |
                                         Location(s)

Recruitment UI --> scoped Candidate API [NOT YET PRODUCTION]
                            |
                       Candidate Core (global; distinct from SystemUser)
                         /                 \
                     CandidateContact     Candidate-Organization Relation [PLANNED]
                            |
                         Resume Core [PLANNED] <-- source-preserving file intake [PLANNED]
                            |
                    AI draft and proposals [RESEARCH]
                            |
                     EscoConcept reference (ESCO 1.2.0)
```

The arrows represent logical relationships, **not** permissions or currently implemented HTTP paths.

## Ownership matrix

| Domain | Canonical owner | Implemented today | Boundaries |
| --- | --- | --- | --- |
| SystemUser, sessions, Organization, Locations | Business API (Express) | Substantial, verified blocks | One business Organization per SystemUser |
| Candidate, contact | Candidate domain in Express (approved schema-only exception) | Prisma foundation in main; local DB pending verification | Globally distinct from SystemUser; Unicode-capable names |
| Candidate–Organization requests and consent | Candidate relation domain | Planned | No global recruiter list; consent and scoped access |
| Resume, evidence, source file | Candidate Resume/intake domains | Research ResumeDraft only | No CV draft promoted to canonical Resume |
| ESCO Knowledge Graph | Independent ESCO reference domain in Express | Version 1.2.0 graph, importer and lexical API | Consumer owns mapping; original facts retained |
| AI processing | AI Core adapter/task boundary | Research experiments, provider-neutral runtime | Cannot grant permission, persist canonical facts or invent concept IDs |
| Recruitment presentation | React/Phoenix | Scaffold and dev-only CV preview | Backend must authorize all data |
| Candidate mobile | Native client | Shell | Separate Candidate authentication in future |

## Approval and verification gates

Do not interpret a planned architecture as a deployed feature. Use explicit statuses and cite code/migration/tests plus owner checks. Candidate schema migration is committed, but local migration history must be reconciled safely. The development CV preview is non-persisting and not a production intake endpoint.
