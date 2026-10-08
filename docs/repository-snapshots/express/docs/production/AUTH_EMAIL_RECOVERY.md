<!-- Archived source from https://github.com/RagueL-HigaBase/higa_systems_express/blob/main/docs/production/AUTH_EMAIL_RECOVERY.md; not current canonical state. -->

# Production Block — Email Verification + Recovery

Status: **DONE**  
Verified and closed: **2026-10-03**

Canonical domain document: `docs/AUTH_AND_SESSION.md`

## Final contract
- one `EmailProvider` abstraction owns transactional delivery;
- Cloudflare Email Sending is the production provider;
- local outbox remains the development provider;
- sender identity comes from `EMAIL_FROM_NAME` / `EMAIL_FROM_ADDRESS`;
- registration verification and resend use the shared provider;
- password reset uses the shared provider;
- organization invitations use the shared provider rather than a dedicated filesystem mailer;
- verification/reset tokens are opaque, hashed in persistence, expiring and single-use;
- resend invalidates previous unused tokens of the same purpose;
- Forgot Password is non-enumerating, including provider delivery failure;
- password reset revokes active sessions;
- bootstrap SYSTEM_OWNER is created email-verified and obtains the first usable password through normal password recovery.

## Verification
Verified locally by the project owner on 2026-10-03:
- backend `npm run verify` passed;
- 19 test files / 85 tests passed;
- password-reset request, token handling, Cloudflare provider behavior and shared-provider mail adapters have automated coverage;
- SYSTEM_OWNER bootstrap/startup passed;
- database migration status reports 8 migrations and up-to-date schema;
- real auth email/recovery flows were manually exercised during the final E2E pass.

Organization invitation delivery remains part of the organization invitation domain for future end-to-end mail scenarios; its shared-provider adapter is covered by the backend automated suite and is not a blocker for this closed auth block.
