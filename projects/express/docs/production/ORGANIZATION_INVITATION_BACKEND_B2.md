# Production Block — Organization Invitation Backend B2

Status: **DONE**  
Verified and closed: **2026-10-04**

Canonical references:
- central `HigaBase_Plans/SYSTEM_ARCHITECTURE.md`;
- `docs/ORGANIZATION.md`;
- `docs/AUTH_AND_SESSION.md`;
- `docs/DATABASE.md`.

## Scope

This block implements the backend contract used by the complete frontend invitation flow.

Endpoints:
- `POST /api/organizations/:organizationId/locations/:locationId/invitations`;
- `POST /api/organizations/invitations/resolve`;
- `POST /api/organizations/invitations/complete`.

## Security and identity invariants

- raw invitation token is never persisted;
- only its cryptographic hash is stored;
- token is single-use;
- default expiry is 72 hours;
- one SystemUser can belong to at most one business Organization;
- target Location must belong to the invitation Organization;
- existing account in another Organization is rejected;
- duplicate Location membership is rejected;
- invitation identity/email are fixed server-side;
- NEW_ACCOUNT invitation possession satisfies email verification;
- EXISTING_ACCOUNT completion requires the existing password;
- completion creates/replaces one locked session for PIN setup;
- backend revalidates everything during completion.

## Transaction boundary

For successful completion one transaction owns:
- invitation claim;
- optional SystemUser creation;
- Organization membership creation when needed;
- Location membership creation;
- invitation ACCEPTED state;
- session replacement/creation.

The invitation is claimed before dependent records are written.

## Delivery

The shared EmailProvider sends a system-owned acceptance URL configured from:
`ORGANIZATION_INVITATION_BASE_URL`.

Editable sender message is not an authorization mechanism.

Delivery failure removes the new pending invitation before the request fails.

## Main Location access

Organization activation and SYSTEM_OWNER bootstrap create/ensure Main Location membership.

Migration:
`20261004152000_backfill_main_location_memberships`

backfills existing Organization members to their Main Location.

## Verification

Project owner reported the final B2 local verification green on 2026-10-04:
- `npx prisma migrate dev` — schema already in sync;
- `npm run db:generate` — Prisma Client generated;
- `npm run verify` — 19/19 test files and 89/89 tests passed;
- TypeScript typecheck passed;
- production build passed;
- `npm run db:status` — 10 migrations found and database schema up to date.

The later resend/revoke/Team lifecycle work is tracked as the next invitation block and does not reopen B2.

## Reopen conditions

Reopen this B2 record only if the create / resolve / complete contract or its security invariants change.

Resend, revoke, Team read model, member status actions and legacy-flow removal belong to the subsequent complete invitation lifecycle block.
