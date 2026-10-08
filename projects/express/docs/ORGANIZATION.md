# Organization and Location Domain

Last reviewed: 2026-10-04

This document is the canonical backend description of organizations, memberships, invitations, legal organization data and reusable Location Profile persistence.

## Core organization identity
`SystemOrganization`
- `hb_name`;
- `hb_vat_number`;
- creator/activation metadata.

The previous Company Profile identity fields `hb_legal_name`, `hb_registration_number` and `hb_organization_type` are no longer part of the active database model.

## Organization legal address
`SystemOrganizationLegalAddress` stores the organization-scoped legal address:
- country;
- city;
- postal code;
- street;
- house number;
- additional.

The legal address is shared organization data. It is editable from the current Location Profile UI because that form exposes Company Legal Address next to the Location Address.

## Locations
`SystemOrganizationLocation`
- organization id;
- location name;
- status;
- timezone;
- `hb_is_main`;
- timestamps.

Current statuses:
- ACTIVE;
- INACTIVE.

The current UI keeps status read-only ACTIVE. Timezone is resolved by the frontend from the browser when no timezone has been persisted yet and is stored on Save.

`SystemOrganizationLocationAddress`
- one physical/location address per Location.

`SystemOrganizationLocationContact`
- email;
- phone;
- website.

`SystemOrganizationLocationSocial`
- LinkedIn;
- Facebook;
- X / Twitter.

Every future Location must reuse this same persistence/API contract by location id.

## Main Location
Every organization receives one Main Location:
- CREATE onboarding creates it together with the organization;
- SYSTEM_OWNER organization bootstrap creates it for a new owner organization;
- the migration seeds one Main Location for every existing organization.

The temporary route key `main` resolves the row where `hb_is_main = true`.
Normal UUID Location ids are also supported for future multi-location navigation.

## Location Profile API
- `GET /api/organizations/:organizationId/locations/:locationId/profile`;
- `PUT /api/organizations/:organizationId/locations/:locationId/profile`.

GET requires:
- authenticated unlocked session;
- membership in the target organization.

PUT requires:
- authenticated unlocked session;
- ADMIN or OWNER membership in the target organization.

PUT validates and transactionally persists:
- Location name;
- timezone;
- Location Address;
- organization Company Legal Address;
- Location contact;
- Location social links.

Status is not writable through the current Location Profile contract.

The API remains explicitly organization-id scoped. Do not replace it with an implicit current/first-organization backend contract.

## Removed Company Profile persistence
The obsolete Company Profile backend structure was removed:
- `SystemOrganizationProfile`;
- `SystemOrganizationContact`;
- `SystemOrganizationBanking`;
- `SystemOrganizationSocial`;
- typed `SystemOrganizationAddress` LEGAL/PHYSICAL model;
- `SystemOrganizationLegal` tax/profile table;
- organization legal-name/registration-number/organization-type profile columns.

Banking fields, tax number, profile description/logo and the old organization-wide contact/social records are not part of the current Location Profile domain.

The migration preserves only data that still belongs to the current domain:
- organization VAT number;
- organization legal address;
- an existing physical address is used to seed the Main Location Address when present, otherwise the legal address seeds it.

Obsolete Company Profile contact/social/banking/profile data are intentionally not migrated into Location records.

## Membership boundary

Cross-project identity and membership invariants are defined by the central HigaBase architecture.

Business invariant:

```text
1 SystemUser
→ maximum 1 Organization
→ zero or more Locations inside that Organization
```

The database now enforces one Organization membership per SystemUser through a unique user membership constraint.

`SystemOrganizationLocationMembership` represents Location access under that Organization membership.

Location membership statuses:
- ACTIVE;
- BLOCKED.

A Location assignment cannot replace or bypass the Organization boundary.

## Membership roles
Organization roles:
- USER;
- ADMIN;
- OWNER.

USER may read Location Profile data.
ADMIN and OWNER may update Location Profile data.

Fine-grained Department/permission architecture is intentionally deferred.

## Creation and activation
CREATE onboarding creates:
- inactive organization;
- VAT number;
- organization legal address;
- Main Location;
- Main Location Address initially matching the legal address.

It does not grant membership immediately.

Only platform SYSTEM_OWNER activates a self-created organization. Activation grants the creator OWNER atomically.

## Invitations

`SystemOrganizationInvitation` is the single invitation entity. Sending an invitation does not create a temporary SystemUser.

Invitation fields include:
- Organization;
- target Location;
- email;
- first name;
- optional middle name;
- last name;
- optional invitation message;
- hashed opaque token;
- role;
- lifecycle status;
- expiry;
- accepted/revoked timestamps;
- inviter;
- timestamps.

Persisted lifecycle states:
- PENDING;
- ACCEPTED;
- REVOKED.

EXPIRED is derived from a PENDING invitation whose `expiresAt` has passed.

Default invitation TTL is 72 hours through `ORGANIZATION_INVITATION_TTL_MINUTES=4320`.

Invitation frontend URL base is configured through:
- `ORGANIZATION_INVITATION_BASE_URL`.

The raw invitation token is delivered only through email and is never stored. Persistence stores only its cryptographic hash.

### Create invitation

Endpoint:
`POST /api/organizations/:organizationId/locations/:locationId/invitations`.

Requires:
- authenticated unlocked session;
- ADMIN or OWNER membership in the target Organization;
- active target Location belonging to that Organization.

Payload:
- firstName;
- middleName nullable;
- lastName;
- email;
- message nullable.

Server rules:
- self-invite is rejected;
- email already bound to another Organization is rejected;
- existing member of the target Location is rejected;
- an unexpired PENDING invitation for the same email/location is rejected;
- existing account in the same Organization reuses its authoritative identity;
- existing account without Organization can be invited without creating a duplicate user;
- OWNER is never granted by this flow; current invitation role is USER.

Invitation delivery uses the shared EmailProvider.

The email contains a backend-owned secure link:
`/invitation/accept?token=<opaque-token>`.

The editable invitation message is content only and never owns or replaces the security link.

If email delivery throws before completion of the request, the just-created PENDING invitation is removed so the caller can retry cleanly.

### Resolve invitation

Endpoint:
`POST /api/organizations/invitations/resolve`.

This endpoint is public and rate-limited.

It validates the opaque token and returns safe invitation context:
- accountState: NEW_ACCOUNT or EXISTING_ACCOUNT;
- fixed First / Middle / Last name;
- email;
- Organization name;
- Location name;
- expiry.

Resolve rejects:
- malformed/unknown token;
- expired invitation;
- already accepted invitation;
- revoked invitation;
- account now belonging to another Organization;
- account already assigned to the target Location.

Resolve is for presentation only. It is not authorization to complete the invitation.

### Complete invitation

Endpoint:
`POST /api/organizations/invitations/complete`.

Completion revalidates the full invitation state and membership constraints.

NEW_ACCOUNT:
- invitation token proves possession of the invited email;
- Password must satisfy the normal minimum length;
- SystemUser is created with invitation identity;
- email is marked verified;
- setup is marked completed;
- Organization membership is created;
- target Location membership is created ACTIVE;
- invitation becomes ACCEPTED;
- first normal session is created locked for PIN setup.

EXISTING_ACCOUNT:
- existing password must be verified;
- no second SystemUser is created;
- an existing membership in another Organization is rejected;
- missing Organization membership is created when valid;
- target Location membership is created ACTIVE;
- missing profile identity fields may be completed from the invitation, but existing identity fields are not overwritten;
- invitation becomes ACCEPTED;
- the previous single session is replaced with a new locked session.

The transaction claims the PENDING invitation before creating user/membership/session records so concurrent reuse cannot commit a partial completion.

### Location membership baseline

`SystemOrganizationLocationMembership` represents user access to Locations inside the user's single Organization.

Statuses:
- ACTIVE;
- BLOCKED.

Organization activation assigns the new OWNER to Main Location.

SYSTEM_OWNER bootstrap also guarantees Main Location membership.

Migration `20261004152000_backfill_main_location_memberships` assigns existing Organization memberships to their Organization Main Location as the compatibility baseline.

### Team read model and lifecycle actions

The Location Team API combines active Location membership rows with still-pending invitation rows without merging the underlying database entities.

Presented Team states:
- PENDING;
- ACTIVE;
- EXPIRED;
- BLOCKED.

Endpoints:
- `GET /api/organizations/:organizationId/locations/:locationId/team`;
- `POST /api/organizations/:organizationId/locations/:locationId/invitations/:invitationId/resend`;
- `POST /api/organizations/:organizationId/locations/:locationId/invitations/:invitationId/revoke`;
- `POST /api/organizations/:organizationId/locations/:locationId/team/:userId/block`;
- `POST /api/organizations/:organizationId/locations/:locationId/team/:userId/activate`.

Resend rotates the opaque invitation token and refreshes the configured invitation TTL.

Revoke transitions a pending invitation to REVOKED.

Location member block/activate changes only the Location membership status and does not remove the user's Organization membership.

A user cannot block their own Location membership through the current action endpoint.

The old signed-in invitation-code accept compatibility endpoint is removed. Invitation acceptance now exists only through the public token resolve/complete flow.

Invitation onboarding remains separate from public Sign Up, Company Setup, Verify Email and Forgot Password.
