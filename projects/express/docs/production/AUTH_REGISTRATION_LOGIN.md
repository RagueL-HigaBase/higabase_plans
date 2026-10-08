# Production Block — Registration + Login

Status: **DONE**  
Verified and closed: **2026-10-03**

Canonical domain document: `docs/AUTH_AND_SESSION.md`

## Final contract
Registration:
- accepts email, password, language and business-use confirmation only;
- always creates a CREATE_ORGANIZATION account;
- collects first/last name only after email verification;
- sends verification through the shared EmailProvider;
- duplicate email returns `409 ACCOUNT_EXISTS`;
- public Join Existing Company is removed.

Login:
- invalid credentials return `401 INVALID_CREDENTIALS`;
- valid unverified accounts return `403 EMAIL_VERIFICATION_REQUIRED`;
- verified accounts without names return `profile-required` with a short-lived onboarding token;
- verified accounts with incomplete company setup return `onboarding-required`;
- established accounts create the normal locked session;
- existing active sessions return `409 SESSION_EXISTS`;
- login rate limiting counts invalid-credential failures only and is isolated by client IP + normalized email;
- successful authentication clears the corresponding failure bucket.

## Verification
Verified locally by the project owner on 2026-10-03 as part of the final auth pass:
- Prisma schema validation passed;
- TypeScript typecheck passed;
- backend test suite passed: 19 files / 85 tests;
- production build passed;
- backend startup passed with SYSTEM_OWNER bootstrap status `already-configured`;
- database migration status reported 8 migrations and `Database schema is up to date!`.
