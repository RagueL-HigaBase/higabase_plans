# Higa Systems Express — Project State

Last reviewed: 2026-10-07

Read with `PROJECT_RULES.md`.

## 1. Current direction
The backend is business/company focused. An isolated experimental ResumeDraft/AI parsing research slice exists, but the production Candidate identity/Resume/relation domain is not implemented or approved in this repository.

Active modules:
- auth;
- onboarding;
- SystemUser profile;
- organizations and reusable Locations;
- platform/SYSTEM_OWNER;
- shared validation/security;
- ESCO Knowledge Graph persistence/import foundation;
- ESCO multilingual runtime prefix search API and optimized search projection;
- provider-neutral AI Core runtime with strict task input/output validation.

## 2. Current API surface
Health:
- `GET /api/health`;
- `GET /api/ready`.

Auth/session:
- registration/login/logout;
- email verification / verification resend;
- Forgot Password / Reset Password;
- PIN setup/unlock/lock/activity;
- `GET /api/me`;
- `PATCH /api/me/availability`.

Onboarding:
- `POST /api/onboarding/profile`;
- `POST /api/onboarding/company-setup`.

SystemUser profile:
- `GET /api/profile`;
- `PUT /api/profile`.

Organization:
- location-scoped invitation create;
- public invitation token resolve;
- invitation completion for NEW_ACCOUNT / EXISTING_ACCOUNT;
- Location Team unified read model;
- invitation resend/revoke;
- Location member block/activate;
- SYSTEM_OWNER organization activation queue/list with creator contact read model;
- SYSTEM_OWNER activation;
- `GET /api/organizations/:organizationId/locations/:locationId/profile`;
- `PUT /api/organizations/:organizationId/locations/:locationId/profile`.

The obsolete organization Company Profile GET/PUT API is removed.

## 3. Current Prisma domain
Identity/session:
- SystemLanguage;
- SystemUser;
- SystemSession with last activity and AVAILABLE/BUSY/AWAY session availability;
- SystemPasswordReset;
- SystemEmailVerification.

SystemUser profile:
- SystemUserProfile;
- SystemUserContact;
- SystemUserAddress;
- SystemUserSocial.

Organization:
- SystemOrganization;
- SystemOrganizationLegalAddress;
- SystemOrganizationLocation;
- SystemOrganizationLocationAddress;
- SystemOrganizationLocationContact;
- SystemOrganizationLocationSocial;
- SystemOrganizationMembership;
- SystemOrganizationLocationMembership;
- SystemOrganizationInvitation.

Platform:
- SystemPlatformMembership.

ESCO reference domain:
- EscoDataset;
- EscoConcept;
- EscoConceptVersion;
- EscoLabel;
- EscoText;
- EscoRelationType;
- EscoRelation;
- EscoReference;
- EscoOccupationSkill;
- EscoHierarchyClosure;
- EscoSearchTerm.

Removed from the active model:
- SystemOrganizationProfile;
- SystemOrganizationContact;
- SystemOrganizationLegal;
- SystemOrganizationBanking;
- SystemOrganizationSocial;
- SystemOrganizationAddress;
- SystemOrganizationType;
- SystemOrganizationAddressType.

## 4. Current migrations
- `20260925130000_business_baseline`;
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
- `20261006170000_resume_draft_research` (experimental).

The Location Profile restructure migration is intentionally destructive for the obsolete Company Profile development structure. It preserves organization VAT/legal address and seeds Main Location rows, then removes the superseded profile/banking/contact/social/address tables and columns.

The latest session migration adds persisted session availability with AVAILABLE as the default. OFFLINE is not persisted as a selectable status; it is derived from the absence of an active authenticated session.

See `docs/DATABASE.md` for migration policy.

## 5. SYSTEM_OWNER
Startup bootstrap guarantees the configured single SYSTEM_OWNER.

The configured SYSTEM_OWNER email is trusted deployment configuration and is marked verified at bootstrap. The initial password is unknown/random; first access uses Forgot Password → Reset Password → Sign In through the shared transactional email provider.

The SYSTEM_OWNER also receives normal OWNER membership for the HigaBase organization administration surface. A newly created owner organization receives a Main Location.

Configuration includes:
- SYSTEM_OWNER_EMAIL;
- SYSTEM_OWNER_FIRST_NAME;
- SYSTEM_OWNER_LAST_NAME;
- SYSTEM_OWNER_ORGANIZATION_NAME.

See `docs/SYSTEM_ADMIN.md`.

## 6. Organization and Location persistence
Organization keeps core company identity required by onboarding/system administration:
- name;
- VAT number;
- legal address;
- activation metadata.

Location Profile persistence stores:
- Location name;
- status;
- timezone;
- Location Address;
- contact;
- website;
- LinkedIn;
- Facebook;
- X / Twitter.

Company Legal Address remains organization-scoped but is read/written through the Location Profile contract.

Organization members may read a Location Profile. ADMIN/OWNER may update it.

The temporary `main` location key resolves the organization's persisted Main Location; UUID Location ids are supported for future locations.

See `docs/ORGANIZATION.md`.

## 7. Validation
Global business-text ASCII validation is active on implemented write flows.

Person names use the stricter person-name validator.

Location Profile validates:
- complete ISO-country addresses;
- email;
- phone;
- URLs;
- IANA timezone.

## 8. Production block status
Closed:
- SystemUser Edit Profile: `docs/production/SYSTEM_USER_PROFILE_EDIT.md` — backend verified and closed on 2026-09-27.
- Organization Activation: `docs/production/ORGANIZATION_ACTIVATION.md` — verified and closed on 2026-09-27.
- Session Presence and Availability: `docs/production/SESSION_PRESENCE_AVAILABILITY.md` — verified and closed on 2026-10-02.

Closed on 2026-10-03:
- Registration + Login: `docs/production/AUTH_REGISTRATION_LOGIN.md` — verified and closed;
- Onboarding: `docs/production/AUTH_ONBOARDING.md` — verified and closed;
- Email Verification + Recovery: `docs/production/AUTH_EMAIL_RECOVERY.md` — verified and closed.

## 9. Known incomplete backend blocks
Closed on 2026-10-04:
- Organization Invitation Backend B2: `docs/production/ORGANIZATION_INVITATION_BACKEND_B2.md` — create/resolve/complete verified and closed;
- Organization Invitation Complete Lifecycle: `docs/production/ORGANIZATION_INVITATION_LIFECYCLE.md` — Team read model, resend/revoke, member block/activate and legacy-flow removal verified and closed.

Closed on 2026-10-06:
- ESCO Knowledge Graph Foundation: `docs/production/ESCO_KNOWLEDGE_GRAPH_FOUNDATION.md`;
- ESCO 1.2.0 Snapshot Importer: `docs/production/ESCO_SNAPSHOT_IMPORTER.md` — real frozen snapshot imported and activated successfully.

Not yet implemented:
- Candidate/Vacancy ESCO references;
- richer Team/member administration beyond current Location membership status actions;
- Location creation/list/delete APIs for additional non-main Locations;
- Location-specific permission overrides;
- production object-storage upload for SystemUser/company assets.

These are known gaps, not hidden TODOs.

## 10. Verification status
AI Core runtime foundation was locally verified on 2026-10-06: strict named/versioned task contracts, provider-neutral adapter boundary, fail-closed input/output validation and stable provider-failure containment are implemented without database or HTTP coupling. Final verification passed 23/23 test files, 106/106 tests, TypeScript and production build.

The ESCO 1.2.0 importer was locally verified on 2026-10-06:
- dataset id 1 activated with status ACTIVE;
- snapshot hash 6239309e7f3c9371d5f0d0d2208b8494d26e171292339c7b1da50fa04cc00077;
- 19,070 concepts;
- 1,040,825 labels;
- 483,734 texts;
- 404,098 relations;
- 542 references;
- 129,004 occupation-skill projection rows;
- 71,652 hierarchy-closure rows;
- 21/21 backend test files and 99/99 tests passed;
- TypeScript and production build passed;
- database status up to date with 11 migrations;
- working tree clean.

ESCO runtime search was subsequently verified with `1,040,825` search terms. For Dutch occupation prefix `operator`, the pre-index SQL plan took `115.331 ms` and scanned `50,001` language candidates, filtering out `49,388`. After `hb_esco_search_terms_prefix_idx`, PostgreSQL directly reached the `613` prefix candidates; SQL execution fell to `3.283 ms`. Warm HTTP measurements fell from roughly `74–78 ms` to roughly `5–6 ms`. The prefix index is therefore a verified runtime requirement, not a speculative optimization. Redis/cache is not required for this autocomplete path at the current measured performance.

Runtime semantic filtering is also verified against the ACTIVE dataset. OCCUPATION search requires source family `occupation`, excluding ISCO taxonomy/classification nodes from ordinary autocomplete. SKILL search requires source family `skill`, excluding ISCED-F nodes. The canonical graph and complete search projection remain unchanged.

Clean deployment rehearsal passed on 2026-10-06 against a new empty PostgreSQL database: all 13 canonical migrations applied successfully via `prisma migrate deploy`; the frozen ESCO 1.2.0 snapshot imported and activated with the expected hash/counts; `npm run db:status` reported the schema up to date; final `npm run verify` passed Prisma validation, TypeScript, 22/22 test files, 102/102 tests, and production build. A server can therefore reproduce the current database/runtime from migrations plus the frozen snapshot without copying the development database.


The Invitation B1 schema foundation was locally verified on 2026-10-04 before B2 implementation:
- migration applied;
- Prisma client generated;
- schema validation passed;
- TypeScript typecheck passed;
- 19/19 backend test files and 85/85 tests passed;
- production build passed;
- migration status was up to date.

Invitation B2 and the complete invitation lifecycle are verified and closed. Final backend verification passed with 19/19 test files, 92/92 tests, TypeScript, production build and 10-migration database status. The project owner also confirmed the invitation lifecycle works correctly end-to-end.

The 2026-10-02 session presence/availability block was locally verified by the project owner.

The final authentication/email block was locally verified and closed on 2026-10-03:
- Prisma schema validation passed;
- TypeScript typecheck passed;
- 19 backend test files / 85 tests passed;
- backend production build passed;
- backend startup passed with SYSTEM_OWNER bootstrap status `already-configured`;
- database migration status reports 8 migrations and an up-to-date schema;
- frontend typecheck and production build passed;
- frontend SonarQube reported 0 issues / 0 effort;
- real authentication/email/PIN recovery behavior was manually exercised during the final pass.

## 11. Candidate Processing audit (2026-10-08)

Code inspection found working experimental AI Core, ResumeDraft JSONB research storage, provider adapters, local PDF/DOCX OpenAI CLI and runtime ESCO search. Production Candidate/Resume Core, provenance-bearing file intake, Candidate-Organization consent/relation, ESCO mappings and screening are still missing. The dev resume routes are not authenticated/organization-scoped and must not be exposed as production APIs. The backend has 14 migration directories; previous 13-migration clean-deployment verification is historical and does not verify the new 14-migration baseline. See `docs/planning/CANDIDATE_PROCESSING_AUDIT_2026_10_08.md` for risks, suggested gates and open decisions. Documentation audit only: no new local verification was performed.

## 12. Domain documentation
- authentication/session: `docs/AUTH_AND_SESSION.md`;
- SystemUser profile: `docs/SYSTEM_USER_PROFILE.md`;
- organizations/locations: `docs/ORGANIZATION.md`;
- SYSTEM_OWNER/platform: `docs/SYSTEM_ADMIN.md`;
- database/migrations: `docs/DATABASE.md`;
- ESCO Knowledge Graph/importer: `docs/ESCO.md`;
- verified production block records: `docs/production/`.


## 2026-10-06 — RESUME-DRAFT-1 verified

The minimal Resume Draft research slice is closed after owner-reported green local verification.

Implemented intentionally as an experimental boundary:
- `ResumeDraftV1` strict Zod contract;
- `hb_resume_drafts` with contract version and JSONB payload;
- Prisma repository;
- read-only development inspection route;
- tests.

This is not the final Candidate/Resume schema. It exists to test heterogeneous real CVs before production normalization.

Next planned experiment: connect `candidate.resume.parse` through AI Core and persist only validated draft output.


## 2026-10-06 — RESUME-PARSE-1 verified

The first real resume parsing vertical slice is owner-verified.

- Local verification: 25 test files / 112 tests passed.
- A live development request successfully executed `candidate.resume.parse@1` through the provider-neutral AI Core and OpenAI Responses adapter.
- The provider output passed strict `ResumeDraftV1` validation before persistence.
- The validated draft was persisted to PostgreSQL and returned with a generated UUID.
- The observed smoke-test extraction correctly separated personal/contact data, languages, professional skills, software/tools, work history, education and certification without inventing achievements.
- AI provider/model details remain outside the ResumeDraft domain payload.
- PDF/DOCX extraction, ESCO normalization, Candidate production persistence and authorization remain outside this closed block.


## 2026-10-07 — OLLAMA-PROVIDER-1 verified

The local/self-hosted provider experiment is owner-verified.

- Local verification passed Prisma validation/generation, TypeScript, 26/26 test files, 115/115 tests and production build.
- A live development request executed the existing `candidate.resume.parse@1` task through `OllamaProvider` and local `gpt-oss:20b`.
- The local model output passed the same strict `ResumeDraftV1` validation used by the OpenAI path and persisted successfully to PostgreSQL.
- Missing optional work-history/education scalar fields were omitted rather than represented as empty strings, so the strict contract remained unchanged.
- Provider selection is configuration-only; ResumeDraft, AiRuntime and the task contract remain provider-independent.
- The local path was visibly slower than the earlier cloud smoke test, but latency has not yet been measured and no performance conclusion is recorded.
- No automatic fallback/router, production local-inference deployment, PDF/DOCX extraction, Candidate production persistence or ESCO mapping was added.
