# Production Block — Organization Invitation Complete Lifecycle

Status: **DONE**  
Verified and closed: **2026-10-04**

Canonical references:
- central `HigaBase_Plans/SYSTEM_ARCHITECTURE.md`;
- `docs/ORGANIZATION.md`;
- `docs/AUTH_AND_SESSION.md`;
- `docs/DATABASE.md`;
- `docs/production/ORGANIZATION_INVITATION_BACKEND_B2.md`.

## Scope

This block completes the operational invitation lifecycle after the verified B2 create/resolve/complete flow.

Implemented:
- unified Location Team read model;
- PENDING / ACTIVE / EXPIRED / BLOCKED presentation states;
- resend invitation;
- revoke invitation;
- Location member block;
- Location member activate;
- self-block protection;
- removal of the legacy signed-in invitation-code accept endpoint.

## API contract

- `GET /api/organizations/:organizationId/locations/:locationId/team`;
- `POST /api/organizations/:organizationId/locations/:locationId/invitations/:invitationId/resend`;
- `POST /api/organizations/:organizationId/locations/:locationId/invitations/:invitationId/revoke`;
- `POST /api/organizations/:organizationId/locations/:locationId/team/:userId/block`;
- `POST /api/organizations/:organizationId/locations/:locationId/team/:userId/activate`.

The existing create/resolve/complete endpoints remain unchanged.

## Domain rules

- Team is a read model; invitation and membership persistence remain separate.
- EXPIRED is derived from an unaccepted/unrevoked PENDING invitation whose expiry has passed.
- resend rotates the opaque token and refreshes the configured TTL.
- revoke transitions only a pending invitation to REVOKED.
- block/activate changes only Location membership status.
- Organization membership is not removed by a Location block.
- the current user cannot block their own Location membership.
- backend remains authoritative for all actions.

## Verification

Automated verification reported by the project owner on 2026-10-04:
- `npm run db:generate` passed;
- Prisma schema validation passed;
- TypeScript typecheck passed;
- 19/19 test files passed;
- 92/92 tests passed;
- production build passed;
- `npm run db:status` reports 10 migrations and an up-to-date database;
- working tree clean.

Manual lifecycle verification was confirmed by the project owner on 2026-10-04:
- invitation creation and acceptance work end-to-end;
- NEW_ACCOUNT / existing-account invitation behavior works as expected;
- Team lifecycle behavior works as expected;
- resend / revoke work as expected;
- block / activate work as expected;
- no issues were reported in the completed invitation flow.

## Reopen conditions

Reopen this production block only if the invitation lifecycle, Team read model, membership status behavior, token security model or related authorization rules change.
