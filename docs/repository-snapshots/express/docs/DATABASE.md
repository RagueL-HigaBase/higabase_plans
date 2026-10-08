<!-- Historical source snapshot from higa_systems_express/main. This is evidence, not canonical current state. For live state see PROJECT_STATE.md in HigaBase Plans. Original: https://github.com/RagueL-HigaBase/higa_systems_express/blob/main/docs/DATABASE.md -->

# Database and Migration Rules

Last reviewed: 2026-10-06

## Naming
All Higa-owned PostgreSQL objects use the `hb_` prefix.

Prisma business/system models use readable `System...` names.

The ESCO reference domain uses readable `Esco...` Prisma model names and `hb_esco_...` physical PostgreSQL names.

## Current migration history
The development migration history was intentionally reset while no production data existed.

Baseline:
- `20260925130000_business_baseline`.

Current appended migrations:
- `20260926134000_system_user_profile`;
- `20260926173000_organization_profile_structure`;
- `20260927143000_system_user_social`;
- `20260930174500_location_profile_structure`;
- `20261002180000_session_availability_status`;
- `20261003093000_email_verification`;
- `20261003130000_registration_profile_split`;
- `20261004133000_invitation_location_membership_foundation`;
- `20261004152000_backfill_main_location_memberships`;
- `20261006113000_esco_knowledge_graph_foundation`;
- `20261006124500_esco_search_projection`;
- `20261006133000_esco_search_prefix_index`;
- `20261006170000_resume_draft_research` (experimental ResumeDraft JSONB, not production Candidate/Resume persistence).

The deleted historical candidate/mobile migration chain must never be restored.

## Local legacy databases
A development database that still contains the removed historical chain must be rebuilt once:

```bash
npx prisma migrate reset --force
npm run db:generate
npm run verify
npm run db:status
```

This is a one-time development reset only.

After a database is aligned with the clean baseline, schema work is append-only through normal Prisma migrations.

## Location Profile restructure
`20260930174500_location_profile_structure` is an intentionally destructive development migration for the obsolete Company Profile domain.

It:
- preserves organization VAT number;
- preserves organization legal address;
- seeds one Main Location for every existing organization;
- seeds the Main Location Address from the old PHYSICAL address when present, otherwise from LEGAL;
- removes obsolete Company Profile profile/contact/banking/social/legal tables and old typed organization addresses;
- removes obsolete company-profile identity columns/enums.

This migration does not delete users, organizations, memberships, invitations, sessions or activation history.

## Session availability
`20261002180000_session_availability_status` appends the persisted session availability enum and `hb_sessions.hb_availability_status`.

Allowed persisted values:
- AVAILABLE;
- BUSY;
- AWAY.

The column defaults to AVAILABLE. OFFLINE is not persisted as an availability value; it is derived by the application from the absence of an active authenticated session.

## Invitation/location membership foundation

`20261004133000_invitation_location_membership_foundation`:
- enforces one Organization membership per SystemUser;
- adds `SystemOrganizationLocationMembership`;
- adds ACTIVE/BLOCKED Location membership status;
- expands the existing Organization invitation with Location, identity, message and lifecycle fields;
- renames the invitation used timestamp to accepted timestamp;
- adds PENDING/ACCEPTED/REVOKED invitation state.

The migration intentionally keeps Location/name fields nullable for legacy development invitation rows. New invitation APIs require complete Location/name context.

`20261004152000_backfill_main_location_memberships` assigns every existing Organization membership to that Organization's Main Location when no such Location membership exists. This establishes the compatibility baseline for the new explicit Location-access model.

## ESCO Knowledge Graph foundation

`20261006113000_esco_knowledge_graph_foundation` adds:
- versioned ESCO dataset persistence;
- stable URI-backed concepts;
- dataset-scoped concept versions;
- multilingual labels/texts;
- canonical typed relations;
- reference metadata;
- occupation-skill and hierarchy-closure projection tables;
- graph traversal/projection indexes.

The deterministic ESCO 1.2.0 importer populates this schema as maintenance data, not as a migration.

The first verified local import activated dataset id `1` with snapshot hash `6239309e7f3c9371d5f0d0d2208b8494d26e171292339c7b1da50fa04cc00077`.

`20261006124500_esco_search_projection` adds the rebuildable `hb_esco_search_terms` runtime projection. The verified active dataset contains `1,040,825` search terms, one per canonical label.

`20261006133000_esco_search_prefix_index` adds the composite B-tree prefix index `(hb_dataset_id, hb_language, hb_normalized_value text_pattern_ops)`. It exists specifically for the runtime prefix autocomplete access pattern. Prisma 7.10 cannot represent this operator-class index in this project's schema DSL, so the index is intentionally SQL-migration-managed and must not be removed merely because it is absent from `schema.prisma`.

## ResumeDraft research migration

`20261006170000_resume_draft_research` adds `hb_resume_drafts` with UUID, version, JSONB payload and timestamps. It does not add a Candidate entity, Resume source/artifact links, ownership, consent, provenance or production intake route. Append future production migrations; do not recast this research table as canonical Candidate storage.

## Clean deployment rehearsal

A historical clean deployment rehearsal passed on 2026-10-06 using a new empty PostgreSQL database. At that point `prisma migrate deploy` applied all 13 then-existing canonical migrations from zero, including the SQL-managed ESCO prefix index. The frozen ESCO 1.2.0 snapshot then imported successfully, passed its count/audit checks, and activated dataset 1. Final `npm run db:status` reported all 13 migrations up to date, and `npm run verify` passed Prisma validation, TypeScript, 22/22 test files, 102/102 tests, and the production build.

Production/server deployment must use `prisma migrate deploy`, not interactive `prisma migrate dev`.

## Verification
For schema-related changes run:
- the appropriate Prisma migration command;
- `npm run db:generate`;
- `npm run verify`;
- `npm run db:status`.

Do not report migration or verification success until the project owner has run the local commands and reported the result.


## Candidate Core v1 schema-only foundation — 2026-10-08

Migration `20261008192000_candidate_core_foundation` adds two isolated tables: `hb_candidates` (global candidate identity and timestamps) and `hb_candidate_contacts` (optional contact row, unique FK to Candidate, with email/phone). Neither contact field is unique, and no Organization owner is embedded in Candidate. No API, consent workflow or audit writer is enabled by this migration. Candidate names are Unicode-capable. Future work: independent CandidateOrganizationRelation/request and append-only CandidateAuditEvent; access through scope-aware services and indexed relation filtering rather than global candidate list. See `HigaBase_Plans/CANDIDATE_ARCHITECTURE.md` §20.
