<!-- Historical main-branch documentation snapshot; not live authority. Original: https://github.com/RagueL-HigaBase/higa_systems_react/blob/main/docs/ORGANIZATION.md. For current shared state consult central PROJECT_STATE.md. -->

# Organization UI

Last reviewed: 2026-10-08

This document is the canonical frontend description of Organization administration and the reusable persisted Location shell.

## Architecture boundary
Organization-level and Location-level UI are separate.

Organization level:
- Dashboard;
- Departments (placeholder);
- Team (placeholder).

Location level:
- one reusable Location shell per `locationId`;
- Location Profile belongs to that Location;
- Company Legal Address remains organization-scoped data exposed inside the Location Profile editor.

The previous standalone Organization Company Profile page/editor is not part of the active frontend.

## Navigation
Current sidebar navigation:

Organization
- Dashboard
- Departments
- Team

Locations
- Main Location
  - Overview
  - Departments
  - Team

Connections, invoices, documents and settings remain available as Location routes and internal workspace sections.

`Main Location` now resolves to a persisted backend Location via the temporary route key `main`. The sidebar label is not hardcoded: it reads the persisted Main Location name from the shared workspace Location context and refreshes immediately after a successful Main Location save.

Future locations must use the same route/component/API structure with their UUID location id. They must not duplicate the Main Location page.

## Organization Dashboard
Active route:
- `/organization/dashboard`.

The Dashboard is intentionally a clean surface while its company-wide content is designed separately.

## Reusable Location routing
Canonical Location routes:
- `/organization/locations/:locationId`;
- `/organization/locations/:locationId/connections`;
- `/organization/locations/:locationId/invoices`;
- `/organization/locations/:locationId/team`;
- `/organization/locations/:locationId/documents`;
- `/organization/locations/:locationId/settings`;
- `/organization/locations/:locationId/profile/edit`.

For Main Location, `locationId` is currently `main`.

## Location shell
The Location page keeps the approved Phoenix Stock Details navigation pattern.

The top tabs own the Location page workspace. Overview content belongs inside the Overview tab; it is not a permanent left-side context column. Member Details uses a different page architecture and must not be used as the structural template for Location Details.

Header:
- persisted Location name;
- small Location subtitle;
- no breadcrumbs;
- no duplicate large page title.

Top tabs:
- Overview;
- Team;
- Operations;
- Finance;
- Compliance;
- Activity.

Working submenu:
- Connections;
- Invoices;
- Team;
- Documents;
- Settings.

The same shell is reused for every Location.

## Location Overview
Overview is now bound to persisted Location Profile data.

Current blocks:
- Location Info;
- Contact;
- Location Address;
- Company Legal Address.

Persisted values shown:
- Location name;
- status;
- timezone;
- phone;
- company email;
- website;
- Location Address;
- Company Legal Address.

Each block uses the compact ellipsis dropdown action pattern:
- Edit;
- Copy to clipboard.

Edit navigates to:
`/organization/locations/:locationId/profile/edit`.


## Location Team
Active route:
- `/organization/locations/:locationId/team`.

The current Team page is a Phoenix-derived table scaffold based on the e-commerce Customers table.

Current UI:
- search;
- Department and Role filter controls;
- More filters;
- Export;
- Add member;
- selectable/sortable/paginated Team table.

The Customers filter tabs (All, New, Abandoned checkouts, Locals, Email subscribers and Top reviews) are intentionally not included.

The table currently exposes only the signed-in real organization member from the authenticated session contract. Name, email, organization role, membership join date, last activity and session availability come from real session/profile data. A complete organization Team read API does not exist yet, so other members, departments, invitations, Export and Add member remain intentionally incomplete.

Member Details for the signed-in user is the canonical Profile view. It combines authenticated session/membership data with the existing SystemUser Profile API. The account dropdown Profile action and the Team row both resolve to the same Member Details route.

All visible Team labels use the existing 28-language business locale system and dates use `Intl.DateTimeFormat` with the active language.

## Location Profile
Active route:
- `/organization/locations/:locationId/profile/edit`.

Layout follows the existing Phoenix-derived profile form pattern.

Left:
- Location Details:
  - Location Name;
  - Status;
  - Timezone;
- Location Address;
- same-as-company-legal-address checkbox;
- conditional Company Legal Address.

Right:
- Contact:
  - Phone;
  - Company email;
  - Website;
- Social & Web:
  - LinkedIn;
  - Facebook;
  - X / Twitter;
- Save at the bottom.

Status is read-only.
Timezone is read-only in the UI. If the backend has no timezone yet, the editor initializes it from the browser timezone and persists it on Save.

## API contract
Load:
- `GET /api/organizations/:organizationId/locations/:locationId/profile`.

Save:
- `PUT /api/organizations/:organizationId/locations/:locationId/profile`.

On successful Save the editor returns to:
`/organization/locations/:locationId`.

The obsolete frontend Organization Company Profile API types, routes, pages and active Organization i18n references are removed.

## Access
The page resolves an ADMIN/OWNER organization membership for editing.

Backend remains authoritative:
- organization member may read a Location Profile;
- ADMIN/OWNER may update it.

A dedicated active-organization selector is still a future UI concern for users with multiple organization memberships.

## Validation
Frontend Zod validates the current Location Profile form:
- required business text;
- ISO country selection;
- address fields;
- phone;
- email;
- URL fields.

Backend validation remains authoritative and also validates the persisted timezone.

## Localization
Location Profile labels use the existing 28-language business locale system.

Country names use `Intl.DisplayNames` with the active language.

## Phoenix rule
The Location shell and editor continue to follow the Phoenix zero-invention rule.

No custom replacement visual system was introduced.

## Final cleanup
Removed as obsolete:
- empty global `/dashboard` route/page;
- empty `OrganizationOverview` placeholder;
- `/organization/profile` and `/organization/profile/edit` compatibility routes;
- `/organization/overview` compatibility route;
- legacy singular `/organization/location...` compatibility routes;
- active Organization/Location references to the old `organizationProfile...` translation namespace.

The verified historical production navigation record is intentionally left unchanged as a record of the earlier state.

## Not yet complete
- full Team backend/domain listing for all organization members, departments, invitations and member administration;
- Organization Dashboard content;
- Location creation/list/delete UI for additional non-main Locations;
- dynamic sidebar population from a future Location list endpoint;
- Location-specific permissions beyond current organization membership roles.
