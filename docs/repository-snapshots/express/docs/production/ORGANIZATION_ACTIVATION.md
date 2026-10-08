<!-- Archived source from https://github.com/RagueL-HigaBase/higa_systems_express/blob/main/docs/production/ORGANIZATION_ACTIVATION.md; not current canonical state. -->

# Production Block — Organization Activation

Status: **DONE**  
Backend verified: **2026-09-27**

Canonical domain document: `docs/SYSTEM_ADMIN.md`

## Scope
This record closes SYSTEM_OWNER organization activation for the CREATE_ORGANIZATION lifecycle:
- real organization activation queue;
- PENDING/ACTIVE status projection;
- SYSTEM_OWNER authorization;
- activation transaction;
- creator OWNER membership;
- pending count used by the frontend indicator.

## Final backend contract
- `GET /api/system/organizations` is available only to SYSTEM_OWNER.
- Status is derived from `hb_activated_at`: PENDING or ACTIVE.
- The read model exposes organization id/name, VAT number, creator email, creator SystemUser contact phone when available, legal country code, creation timestamp and pending count.
- Default order is oldest PENDING first, then newest ACTIVE.
- Activation remains transactional: activation metadata is stored and the original creator receives OWNER membership in the same transaction.
- Repeated activation returns the already-active result.
- No separate INACTIVE status or delete workflow exists in this block.

## Verification
Verified locally by the project owner:
- `npm run db:generate`;
- `npm run verify`;
- `npm run db:status`.

Observed result:
- Prisma Client generated successfully;
- Prisma schema valid;
- TypeScript typecheck passed;
- 12 test files passed;
- 57 tests passed;
- production build passed;
- database schema is up to date.

## Reopen only if
Reopen this block if:
- organization activation authorization changes;
- activation transaction semantics change;
- activation statuses change;
- creator OWNER assignment changes;
- SYSTEM_OWNER activation queue contract changes.

Final table-read refinement verified after closure: creator email/phone are included in the SYSTEM_OWNER activation queue without schema migration. Otherwise treat backend organization activation as closed.
