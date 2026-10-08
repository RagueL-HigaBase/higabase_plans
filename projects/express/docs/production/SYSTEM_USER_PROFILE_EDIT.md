# Production Block — SystemUser Edit Profile

Status: **DONE**  
Backend verified: **2026-09-27**

Canonical domain document: `docs/SYSTEM_USER_PROFILE.md`

## Scope
This record closes authenticated SystemUser profile persistence and account-security boundaries:
- personal identity;
- contact;
- address;
- social links;
- account email/password separation;
- transactional profile persistence.

## Final backend contract
- `GET /api/profile` returns personal/contact/address/social profile data.
- `PUT /api/profile` transactionally persists SystemUser identity, SystemUserProfile, SystemUserContact, SystemUserAddress and SystemUserSocial.
- Social links are optional LinkedIn, Facebook and X / Twitter URLs.
- Social data uses the dedicated 1:1 `SystemUserSocial` aggregate and `hb_user_social` table.
- Email/password changes remain outside profile PUT and use the authenticated account endpoint with current-password verification.
- Profile photo persistence remains object-storage-key based; upload storage is not implemented in this block.

## Verification
Verified locally by the project owner:
- `npx prisma migrate dev`;
- `npm run db:generate`;
- `npm run verify`;
- `npm run db:status`.

Observed result:
- Prisma schema in sync;
- Prisma Client generated successfully;
- Prisma schema validation passed;
- TypeScript typecheck passed;
- 12 test files passed;
- 58 tests passed;
- production TypeScript build passed;
- 4 migrations found;
- database schema is up to date.

## Reopen only if
Reopen this block if:
- SystemUser profile aggregate structure changes;
- social-link persistence changes;
- account/profile security boundary changes;
- profile photo object-storage implementation is added.

Otherwise treat backend SystemUser Edit Profile as closed.
