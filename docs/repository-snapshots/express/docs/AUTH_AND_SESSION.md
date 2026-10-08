<!-- Historical source snapshot from higa_systems_express/main. This is evidence, not canonical current state. For live state see PROJECT_STATE.md in HigaBase Plans. Original: https://github.com/RagueL-HigaBase/higa_systems_express/blob/main/docs/AUTH_AND_SESSION.md -->

# Authentication and Session

Last reviewed: 2026-10-04

This document is the canonical backend description of business-account registration, email verification, authentication, onboarding, password recovery, sessions and PIN behavior.

## Account identity
`SystemUser` is the business-account identity. It stores:
- email;
- first, optional middle and last name;
- password hash;
- language;
- registration intent;
- setup-completion state;
- email-verification state.

Public self-registration never grants organization or platform privileges.

Public registration always starts a new organization account. Joining an existing organization is invitation-only.

## Public registration flow

```text
Create Company Account
→ Verify Email
→ Complete Profile
→ Create Company
→ Set PIN
→ Under Review
```

Initial registration accepts:
- email;
- password;
- languageCode;
- business-use confirmation.

Email is trimmed and normalized to lowercase. Password length is 8-256 characters.

First/last name are intentionally collected only after email verification. Person names follow the ASCII passport-name rule.

A newly registered account cannot sign in until its email is verified. Login returns `EMAIL_VERIFICATION_REQUIRED` for a valid unverified account.

Successful verification consumes a single-use verification token and returns a short-lived onboarding token with `profile-required` state so the user can continue directly without another login.

If a verified user later signs in before profile completion, login returns the same `profile-required` recovery state.

## Email delivery
All transactional account mail is routed through the shared `EmailProvider` abstraction.

Current provider modes:
- `EMAIL_PROVIDER=cloudflare` — production Cloudflare Email Sending;
- `EMAIL_PROVIDER=local` — development filesystem outbox.

Shared sender configuration:
- `EMAIL_FROM_NAME`;
- `EMAIL_FROM_ADDRESS`.

The same provider is used for:
- registration verification;
- verification resend;
- password reset;
- organization invitations.

Provider-specific transport must not leak into auth or organization business logic.

## Invitation account creation

Joining an existing Organization is invitation-only and does not reuse public registration or password recovery.

New invited account:

```text
Invitation email
→ /invitation/accept?token=...
→ token resolve
→ fixed identity + Organization + Location
→ Password / Confirm Password
→ complete invitation
→ SystemUser + Organization membership + Location membership
→ verified email
→ locked session
→ PIN setup
→ workspace
```

The invitation token proves possession of the invited email, so no second email-verification flow is required.

Existing account:
- token resolve reports EXISTING_ACCOUNT;
- current password is required;
- no duplicate SystemUser is created;
- the account must be unassigned or already belong to the same Organization;
- completion adds only the required Organization/Location access;
- a new locked session is issued for the normal PIN flow.

Invitation completion uses the same password hashing and session-token primitives as normal authentication but remains a separate business flow.

## Password reset
Forgot Password is deliberately non-enumerating:
- invalid input, unknown account and accepted requests expose the same public response;
- provider delivery failure is logged server-side but does not expose account existence.

For a known account:
1. all previous unused reset tokens are invalidated;
2. a cryptographically opaque token is generated;
3. only its hash is stored;
4. the reset link points to frontend `/reset-password?token=...`;
5. the token expires according to `PASSWORD_RESET_TTL_MINUTES`;
6. the token is single-use.

Successful reset:
- replaces the password hash;
- consumes the reset token;
- removes all active sessions;
- requires a normal Sign In afterward.

## SYSTEM_OWNER first access
Startup bootstrap guarantees one configured `SYSTEM_OWNER`.

The deployment-configured `SYSTEM_OWNER_EMAIL` is trusted and the bootstrap identity is created already email-verified. A random unknown bootstrap password is stored only as a hash.

First access is therefore:

```text
Server bootstrap
→ Forgot Password
→ real reset email
→ Reset Password
→ Sign In
```

The SYSTEM_OWNER also receives normal OWNER membership for the configured HigaBase organization administration surface.

## Pre-session onboarding
After email verification, profile completion is required before company setup.

The short-lived onboarding token is signed and bound to:
- SystemUser id;
- CREATE_ORGANIZATION intent;
- expiry.

Profile completion stores first/last name and keeps the same onboarding token for the company-setup step.

Company setup:
- collects company name, VAT and legal address;
- creates an inactive organization;
- records its creator;
- marks setup complete;
- creates the first normal session in locked state.

Until SYSTEM_OWNER activation grants membership, the new organization has no workspace access.

## Session
Current invariants:
- one session per user;
- 8-hour lifetime;
- session token stored as a hash;
- first session starts locked;
- PIN is exactly 4 digits;
- 15-minute idle lock;
- second invalid PIN attempt revokes the session;
- password reset removes all active sessions;
- presence and availability are separate;
- Online is derived from an authenticated session;
- Offline is derived from the absence of an active session;
- availability is AVAILABLE, BUSY or AWAY.

Established organization membership or platform role remains authoritative workspace access.

## Current session contract
`GET /api/me` returns:
- SystemUser id;
- firstName / lastName;
- email;
- languageCode;
- registrationIntent;
- setupCompleted;
- organizations with organization role;
- platformRole;
- workspaceAvailable;
- lastActivityAt;
- availabilityStatus;
- locked;
- pinConfigured.

Backend access data is authoritative. Frontend must not manufacture roles or workspace access.

## Rate limits
Current process-local limits include:
- registration: 5 / 15 minutes;
- invitation resolve: 20 / 15 minutes;
- invitation completion: 10 / 15 minutes;
- login: 3 failed credential attempts / 10 minutes per client IP + normalized email; successful authentication clears that account/client failure bucket;
- forgot password: 5 / 15 minutes;
- reset-password submission: 3 / 15 minutes;
- email-verification resend: 5 / 15 minutes;
- verification submission: 10 / 15 minutes;
- account credential change: 3 / 10 minutes.

The process-local limiter is acceptable only for the current single-backend-instance deployment. Shared rate-limit storage is required before horizontal scaling.

## Production records
- Registration + Login: `docs/production/AUTH_REGISTRATION_LOGIN.md`.
- Onboarding: `docs/production/AUTH_ONBOARDING.md`.
- Email Verification + Recovery: `docs/production/AUTH_EMAIL_RECOVERY.md`.
