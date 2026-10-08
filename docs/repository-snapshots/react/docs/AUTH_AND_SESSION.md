<!-- Historical main-branch documentation snapshot; not live authority. Original: https://github.com/RagueL-HigaBase/higa_systems_react/blob/main/docs/AUTH_AND_SESSION.md. For current shared state consult central PROJECT_STATE.md. -->

# Authentication and Session UI

Last reviewed: 2026-10-08

This document is the canonical frontend description of authentication, onboarding, session and PIN flows.

## Routes
- `/sign-in`
- `/sign-up`
- `/forgot-password`
- `/reset-password`
- `/verify-email`
- protected root `/`

## Account flow
Registration creates the business account but does not authenticate it. Sign Up directs to email verification. The missing-email link on Sign In opens the verification page, which offers a resend request.

Login follows backend state:
- unverified email -> Verify Email/resend;
- verified account with incomplete identity -> Complete Profile using a short-lived onboarding token;
- completed profile without established access -> in-memory Company Setup;
- established account -> normal session/PIN flow.

CREATE:
Sign Up -> Verify Email -> Complete Profile -> Company Setup -> locked session -> PIN -> standalone pending-activation page. No workspace navbar, sidebar, footer or profile access is exposed before activation creates membership.

JOIN:
Sign Up -> Verify Email -> Complete Profile -> Company Setup -> locked session -> PIN -> standalone invitation page. No workspace shell is exposed before invitation acceptance creates membership.

Existing organization/platform member:
Sign In -> session/PIN -> workspace.

SYSTEM_OWNER:
bootstrap account -> Forgot Password -> Reset Password -> Sign In -> session/PIN -> workspace.

## Session authority
`AuthGate` and `AuthSessionContext` consume the backend session contract.

The account dropdown Profile action resolves to the signed-in user's organization Member Details route. The obsolete standalone read-only `/profile` page is removed; `/profile/edit` remains the profile editor.

Frontend must not invent:
- organization memberships;
- organization roles;
- platform roles;
- workspace availability.

## PIN
PIN setup/unlock follows backend rules. The frontend is not the security authority. Four digit inputs are rendered as obscured password fields while retaining numeric input mode and the previously approved Phoenix PIN layout. Do not regress to visibly displayed digits.

## Presence and availability
Presence and availability are separate:
- Online/Offline is system-derived from authenticated session existence;
- Available/Busy/Away is user-selectable session availability;
- Offline is never offered as a manual selection;
- a new session defaults to Available;
- the account dropdown persists availability through `PATCH /api/me/availability` and then refreshes the current session;
- the account avatar mirrors availability using the existing Phoenix Avatar status variants.

Member Details shows presence and availability together in the Currently row. Last seen remains a separate value from `lastActivityAt`.

## Company Setup
CREATE collects:
- company name;
- VAT number;
- legal address.

JOIN confirms the existing-company intent.

The current backend registration contract initially records email/password; it does not accept a registrationIntent selection. Company Setup receives the backend-authoritative onboarding intent. It must not derive authorization from a frontend-only selection.

The onboarding token remains in component memory only. If it expires during Company Setup, the frontend discards the pending setup state and returns the user to Sign In to obtain a fresh token.

## Internationalization
All visible auth/onboarding/session UI uses the existing 28-language business locale packages.

## Closed production records
- Onboarding pre-workspace flow: `docs/production/AUTH_ONBOARDING.md`.
