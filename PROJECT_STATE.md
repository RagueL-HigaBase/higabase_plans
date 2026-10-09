# HigaBase — Cross-Repository Project State

**Checkpoint:** 2026-10-08. **Authority:** summary of project docs and owner-reported local checks. This is not a live automated inventory.

## Nonnegotiable development constraint

**Work only on existing `main` in each active repository. Do not create/switch to feature branches or open PRs.** Existing non-main branches are retained for later audit and cleanup **after Candidate + ESCO work is closed**. No bulk merges, destructive resets, or deletions.

## Repository responsibilities

| Repository | Responsibility | Current observation |
| --- | --- | --- |
| HigaBase_Plans | Cross-system rules and this consolidated state | Canonical documentation hub |
| higa_systems_express | Business API, ESCO, isolated Candidate schema foundation | Local main synced to `0173196`; owner ran Prisma generate and verify: 34 test files / 141 tests passed, typecheck/build green (2026-10-08) |
| higa_systems_react | Phoenix business web UI | Main has restored PIN/email verification, Organization/Locations and static Candidate Prototype; visual owner check remains VERIFY |
| higa_systems | Older combined System Core | Legacy implementation; do not treat as the current React/Express pair |
| higa_systems_native | Mobile candidate shell | Foundation only, no current backend integration |
| higa-esco-extractor | Frozen ESCO extraction | Reference snapshot pipeline, not Candidate application |

## Implemented / historically verified

- Auth, PIN/session, registration/email recovery, company onboarding, organization activation and invitations: documented closed backend blocks.
- Express ESCO 1.2.0 import/search knowledge foundation: earlier owner-verified ACTIVE snapshot; 19,070 concepts, 1,040,825 labels, 404,098 relations, 129,004 occupation-skill rows. Frozen reference; no speculative update.
- AI Core is provider-neutral with strict contracts; OpenAI and local Ollama experiments, ResumeDraft research and PDF/DOCX tests exist.
- React Recruitment Candidates list/preview and restored legacy UX exist on main; some local UI verification still pending.

## Current critical path (do not skip gates)

1. **Candidate Core schema-only** already landed in Express main: `Candidate`, `CandidateContact`, migration `20261008192000_candidate_core_foundation`. This is **not** permission to expose Candidate APIs or populate candidate PII.
2. **Local migration reconciliation blocked:** On 2026-10-08 owner-reported `npm run db:status` lists 15 repo migrations and mismatch with local Windows PostgreSQL: applied `20261006145754_candidate` is absent from repo; legitimate candidate foundation migration not yet applied. The former migration file contained `DROP INDEX "hb_esco_search_terms_prefix_idx"`. Determine actual SQL application/index presence before modifying DB history. No reset, unsafe delete, or automatic deploy.
3. Candidate–Organization relation/request/consent, scoped read/write and audit: not yet production-ready.
4. Source artifact/provenance and safe durable CV intake: not yet production-ready. The development preview must remain non-persisting and non-production.
5. Resume Core and AI draft validation-to-review boundary.
6. Candidate ESCO occupation/skill mapping as **proposals** referencing real `EscoConcept` while retaining original CV facts.
7. Verification/screening and authorized Recruitment frontend end-to-end.

## Open checks

- Express: migration reconciliation and DB-level confirmation; 141 tests green does not verify the local DB migration state.
- React: `npm run verify` on owner local main and browser review of restored PIN, verification, Organization/Locations, Candidate Prototype and preview flow.
- Git: 19 additional branches across three repositories were seen in the 2026-10-08 audit; defer deletion until above Candidate + ESCO milestones close. Compare diffs before cleaning.
- Old higa_systems repository requires an explicit active-vs-legacy ownership decision before cleanup.
- Some local PROJECT_STATE and planning notes predate schema and UI updates. Historical docs are evidence, not new authorization.

## Core architectural invariants

- `SystemUser` and `Candidate` identities are different; a business SystemUser belongs to at most one business Organization, with multiple Locations in that Organization.
- Candidate is global and may relate to several Organizations only via separately governed relation/consent.
- Resume is not Candidate Profile; AI Draft is neither canonical Candidate nor Resume.
- Original CV/source text remains; ESCO mappings are additive and dataset-resolved; inferred skills require explicit handling.
- AI never owns persistence, auth, canonical identity or approval.
- React uses Phoenix UI and existing localization, not invented independent styling.

## Completion protocol

A block is **DONE** only after code, tests, required migrations, owner verification, and central docs agree. Update this state immediately after every approved verified block; move transient detail out of local CURRENT_TASK only after closing it.

## Regression risk checkpoint — 2026-10-08 (OPEN / NOT VERIFIED)

The owner reports that earlier branch merges or branch switching may have displaced UI behavior and data in parts of the system. This is a **reported risk**, not a proven inventory of lost records or a completed repair. React main previously required recovery of masked PIN, email verification, Candidate Prototype and Organization/Locations navigation. Full functional audit has **not** been performed.

**Deferred audit after current approved Candidate + ESCO blocks:** start at Sign In, registration/verification/recovery, PIN/session, onboarding, Organization/Location, System Admin, Recruitment/Candidates and Candidate Prototype; trace route → UI → API → authorization → database, compare central expected contracts with observable main behavior, identify regression provenance where possible, record screenshots/tests/defects and fix only in separately approved narrow blocks. Audit actual data integrity without destructive repair. Follow with SonarQube scan/remediation and explicit closure criteria. Do not call this DONE until owner confirmation.

**Nonnegotiable agent execution contract:** [AGENT_RULES.md](AGENT_RULES.md). Implement only the owner's scope; any architecture extension needs explicit approval. Main only, no new branches or PRs.

## Candidate development factual checkpoint — 2026-10-09

See [end-of-day Candidate Development Pipeline checkpoint](docs/operations/CANDIDATE_DEV_CHECKPOINT_2026_10_09.md). Later owner evidence supersedes earlier Candidate-only verification gaps in this document: local PostgreSQL 16/16 migrations current; Express 35 test files / 150 tests plus typecheck/build successful; Candidate dev upload/save/list/detail exercised through browser and pgAdmin. The production Candidate consent/access/source-artifact/ESCO boundaries **remain unresolved**. Reset interactive count (4 experimental candidates) and cancellation confirmed, but destructive deletion not tested. This section does not close older unrelated audit/migration-history concerns beyond the reported current database status.
