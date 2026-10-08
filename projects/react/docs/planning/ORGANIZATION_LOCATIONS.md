# Organization Locations

Status: PLANNED

## Purpose
Reusable multi-location structure for an Organization.

## Core rule
A Location is an instance of one shared route/component architecture.

Future locations must be addressed by `locationId` and must not be implemented by copying Main Location pages.

Canonical route family:
- `/organization/locations/:locationId`;
- `/organization/locations/:locationId/profile/edit`;
- Location workspace child routes under the same `:locationId`.

## Current development instance
`Main Location` is temporary frontend-only scaffolding using id `main`.

No Location database entity or persistence exists yet.

## Shared Location shell
Every Location is expected to reuse:
- Overview;
- Team;
- Operations;
- Finance;
- Compliance;
- Activity;
- Connections;
- Invoices;
- Team workspace;
- Documents;
- Settings;
- Location Profile.

## Location Profile
Location Profile is Location-scoped.

Do not reuse the removed Organization Company Profile form as the Location model.

Fields, validation, persistence and RBAC must be designed in explicit future blocks.

## Not decided yet
- persisted Location schema;
- creation flow;
- location list/selector behavior;
- default/Main Location semantics after persistence exists;
- location-level permissions;
- final ownership/source for organization-level legal data shown inside Location UI.
