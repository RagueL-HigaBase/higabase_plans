<!-- Historical main-branch documentation snapshot; not live authority. Original: https://github.com/RagueL-HigaBase/higa_systems_react/blob/main/PROJECT_STATE.md. For current shared state consult central PROJECT_STATE.md. -->

# Higa Systems React — Project State

Last reviewed: 2026-10-08

Read with `PROJECT_RULES.md`.

## 1. Current direction
The active frontend is business/company only.

The application uses the Phoenix workspace shell and approved Phoenix-derived business pages.

Organization-level navigation and Location-level navigation are separate concepts:
- Organization uses a Dashboard surface;
- each Location reuses one Location shell keyed by `locationId`.

## 2. Active routes
Public/auth:
- `/sign-in`;
- `/sign-up`;
- `/forgot-password`;
- `/reset-password`;
- `/verify-email`.

Protected:
- `/`;
- `/profile/edit`;
- `/dashboard`;
- `/recruitment`;
- `/recruitment/candidates`;
- `/organization/departments`;
- `/organization/team`;
- `/organization/dashboard`;
- `/organization/locations/:locationId`;
- `/organization/locations/:locationId/connections`;
- `/organization/locations/:locationId/invoices`;
- `/organization/locations/:locationId/departments`;
- `/organization/locations/:locationId/team`;
- `/organization/locations/:locationId/team/:memberId`;
- `/organization/locations/:locationId/documents`;
- `/organization/locations/:locationId/settings`;
- `/organization/locations/:locationId/profile/edit`;
- `/system/organizations`;
- `/system/candidate-prototype` with dashboard/resume children.

Development/reference:
- `/system/typography` — temporary A1/A2 information-pattern reference surface.

Unknown routes redirect to `/`.

Obsolete compatibility routes for the former Organization Profile and singular `/organization/location...` path family were removed during the Organization final cleanup.

## 3. Authentication/session
Implemented:
- registration and email verification/resend;
- profile completion after verified email;
- sign-in;
- Company Setup onboarding with registration intent fixed from registration;
- PIN setup/unlock with obscured digit inputs;
- idle-session handling through backend state;
- standalone pending organization activation before workspace access;
- standalone organization invitation acceptance before workspace access;
- sign-out;
- persisted session availability: Available / Busy / Away;
- system-derived Online / Offline presence;
- localized auth/session UI.

See `docs/AUTH_AND_SESSION.md`.

## 4. SystemUser profile
`/profile/edit` is protected by workspace availability and is not exposed to pre-workspace users.

It loads through `GET /api/profile` and saves personal/contact/address/social data through transactional `PUT /api/profile`. Account email/password use the separate authenticated account endpoint.

The page uses the approved two-column form layout: Identity/Address/Social & Web/Profile Picture on the left, Account/Personal/Contact/Change Password on the right. Social links are LinkedIn, Facebook and X / Twitter.

Photo remains preview-only until object storage exists.

See `docs/SYSTEM_USER_PROFILE.md`.

## 5. Organization and Locations
Current Organization navigation exposes Dashboard, Departments and Team. A separate Locations section exposes Main Location (Overview, Departments, Team). Global Dashboard and Recruitment/Candidates remain separate sections.

The old frontend Company Profile presentation page/editor and their compatibility routes have been removed. Active Organization/Location components no longer reference the legacy `organizationProfile...` i18n namespace.

`/organization/dashboard` is now the Organization-level surface. It intentionally contains no legacy Company Profile presentation content while the Dashboard is designed as a separate future block.

Main Location now resolves to a persisted backend Location through the temporary route key `main`. Future UUID Location ids reuse the same route and API contract.

Every Location uses the same reusable route/component shell:
- Overview;
- Team;
- Operations;
- Finance;
- Compliance;
- Activity.

The Location working submenu exposes:
- Connections;
- Invoices;
- Team;
- Documents;
- Settings.

The Team route renders a Phoenix Customers-derived table scaffold with search, Department/Role filters, More filters, Export, Add member, row selection, sorting and pagination. Customer filter tabs are intentionally omitted.

The obsolete Anna Carry mock dataset is removed. The current table contains the signed-in real member assembled from the authenticated session contract. Member Details for that user combines session/membership values with the SystemUser Profile API. Full organization-member listing and member administration still require a dedicated Team backend/domain API.

Location Overview loads persisted Location Profile data from the backend and lives inside the Overview top-tab workspace. Its compact ellipsis Edit actions navigate to:
`/organization/locations/:locationId/profile/edit`.

The Location Profile route is now connected to the backend Location Profile API. It loads/saves Location Details, Location Address, Company Legal Address, Contact and Social & Web links. Status remains read-only. If timezone is not yet stored, it is derived from the browser and persisted on Save. Successful Save returns to the current Location Overview.

Future persisted Locations must reuse the same route/component structure by `locationId`; new locations must not duplicate page code.

The obsolete backend Organization Profile contract has been replaced by the organization-scoped Location Profile contract. Frontend Company Profile API types are removed.

See `docs/ORGANIZATION.md`.

## 6. SYSTEM_OWNER
SYSTEM_OWNER receives:
- the normal Organization administration surface for its HigaBase OWNER membership;
- a separate System section.

`/system/organizations` uses the approved Phoenix CRM Leads table scaffold and loads real organizations from the SYSTEM_OWNER backend activation queue. The first cell shows organization name with VAT + PENDING/ACTIVE badge below it; creator Email and Phone are separate columns, followed by Country and Create date/time. Pending organizations are surfaced first and can be activated from the row action menu. The Control Panel indicator is driven by the real pending count.

See `docs/SYSTEM_ADMIN.md`.

## 7. Phoenix UI
The workspace shell, SystemUser Edit Profile form, Location shell/navigation and system table follow the zero-invention Phoenix rule.

Higa A1/A2 information typography is a local reusable overlay through `HigaInfoSurface`; it does not override Phoenix global CSS. Member Details and Location Overview reuse the typography while retaining different page architectures.

See `docs/PHOENIX_UI.md`.

## 8. Internationalization
The existing 28 auth/business locale packages are the active localization source.

Organization Dashboard, Location Profile and Team template labels are present in all 28 business locales.

Country names use `Intl.DisplayNames` with the active language where implemented.

## 9. ASCII validation
Shared frontend helpers filter unsupported non-ASCII business input on implemented persisted forms.

SystemUser profile Zod validation remains active.

The removed Company Profile editor no longer contributes frontend validation. Backend validation remains authoritative for the Location Profile API.

## 10. Closed production blocks
- SystemUser Edit Profile: `docs/production/SYSTEM_USER_PROFILE_EDIT.md` — frontend verified and closed on 2026-09-27.
- Organization Activation: `docs/production/ORGANIZATION_ACTIVATION.md` — frontend verified and closed on 2026-09-27.
- Onboarding pre-workspace flow: `docs/production/AUTH_ONBOARDING.md` — frontend verified and closed on 2026-09-27.
- Organization Location Navigation: `docs/production/ORGANIZATION_LOCATION_NAVIGATION.md` — historical navigation block verified on 2026-09-30; later Location routing/persistence changes were locally verified as part of the 2026-10-01 frontend verification pass.
- Frontend Architecture Stabilization: `docs/production/ARCHITECTURE_STABILIZATION_2026_10_02.md` — verified and closed on 2026-10-02.

## 11. Known incomplete frontend blocks
Not yet complete:
- full Team backend/domain listing for all members, departments, roles/access, invitations and actions;
- Organization Dashboard content/domain;
- Location creation/listing and real multi-location navigation;
- production SystemUser/company image upload persistence;
- explicit active-organization selection for users with multiple organization memberships;
- System Overview/Users/Requests routes.

## 12. Verification status
The 2026-10-01 frontend baseline was locally verified by the project owner:

```bash
npm run verify
```

Result:
- TypeScript typecheck passed;
- production Vite build passed;
- SonarQube re-scan reported 0 issues / 0 effort.

Non-blocking build warnings remain:
- third-party `lottie-web` contains `eval`;
- some generated chunks exceed Vite's default size warning threshold.

The 2026-10-01 SonarQube maintenance pass changed code quality/maintainability only and did not intentionally change product behavior.

The 2026-10-02 architecture stabilization pass was locally verified by the project owner. TypeScript typecheck and Vite production build passed. The known non-blocking `lottie-web` eval warning and Vite chunk-size warning remain.

## 13. Domain documentation
- authentication/session: `docs/AUTH_AND_SESSION.md`;
- SystemUser profile: `docs/SYSTEM_USER_PROFILE.md`;
- Organization/Location architecture: `docs/ORGANIZATION.md`;
- SYSTEM_OWNER/system admin: `docs/SYSTEM_ADMIN.md`;
- Phoenix integration: `docs/PHOENIX_UI.md`;
- future Organization Dashboard/Locations planning: `docs/planning/`;
- verified production block records: `docs/production/`.

## Workspace recovery checkpoint — 2026-10-08

Read `docs/planning/WORKSPACE_RECOVERY_2026_10_08.md`. The previous owner-local UI differed from the old main history. Missing PIN masking, email verification/recovery, Candidate Dossier Prototype and split Organization/Locations navigation have been restored to main. The Candidate Prototype is static test content and is not linked to real Candidates. Browser verification remains pending the owner.
