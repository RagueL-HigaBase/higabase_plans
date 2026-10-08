# Organization Team

Status: UI TEMPLATE

## Purpose
Location-scoped internal team surface for future organization members, departments, roles/access and invitations.

Members are internal organization users, not candidates.

## Current frontend template
Route:
- `/organization/locations/:locationId/team`.

The page currently uses the Phoenix e-commerce Customers table as the structural/visual reference.

Implemented as UI-only:
- Team heading;
- search;
- Department filter;
- Role filter;
- More filters button;
- Export action;
- Add member action;
- Phoenix advance table;
- row selection;
- sorting;
- pagination;
- localized table headers and demo role/department/status/date values.

Explicitly omitted from the Phoenix Customers reference:
- All;
- New;
- Abandoned checkouts;
- Locals;
- Email subscribers;
- Top reviews.

The current rows are frontend mock data only.

## Not connected yet
Do not treat the template as implemented Team domain behavior.

Not yet connected:
- backend users/memberships;
- Location membership;
- departments;
- role/access persistence;
- invitations;
- Export behavior;
- Add member behavior;
- filter behavior beyond the existing visual controls.

Those contracts require separately approved backend/domain work.
