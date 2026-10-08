<!-- Archived source from https://github.com/RagueL-HigaBase/higa_systems_express/blob/main/docs/production/AUTH_ONBOARDING.md; not current canonical state. -->

# Production Block — Onboarding

Status: **DONE**  
Verified and closed: **2026-10-03**

Canonical domain document: `docs/AUTH_AND_SESSION.md`

## Final contract

```text
Verify Email
→ Complete Profile
→ Create Company
→ Set PIN
→ Under Review
```

Joining an existing company is invitation-only.

Backend behavior:
- successful email verification returns a short-lived signed onboarding token;
- the token is bound to the verified SystemUser and CREATE_ORGANIZATION intent;
- Complete Profile stores first/last name using that token;
- the same onboarding token continues into company setup;
- company setup cannot run before profile completion;
- company setup creates the inactive organization and first locked normal session;
- no workspace access exists until established organization/platform membership exists;
- reused/expired/tampered onboarding tokens are rejected;
- interrupted verified accounts recover through Sign In and resume at the backend-required step.

## Verification
Verified locally by the project owner on 2026-10-03:
- Prisma validation passed;
- TypeScript typecheck passed;
- 19 test files / 85 tests passed;
- production build passed;
- database schema is up to date with 8 migrations;
- fresh auth/onboarding behavior was manually exercised during the final auth pass.
