# Session Presence and Availability

Verified: 2026-10-02

## Scope
Closed the backend session metadata and availability block introduced during the 2026-10-02 architecture pass.

## Final contract
- `GET /api/me` exposes `lastActivityAt`, organization membership `joinedAt`, and `availabilityStatus`.
- Persisted availability values are AVAILABLE, BUSY, and AWAY.
- AVAILABLE is the default for a new session.
- `PATCH /api/me/availability` updates availability for an authenticated unlocked session.
- OFFLINE is not persisted and is not user-selectable; it is derived from the absence of an active authenticated session.
- SessionRepository requires availability persistence.
- Prisma migration `20261002180000_session_availability_status` is part of the active migration chain.

## Verification
Project owner verified locally:

```bash
npx prisma migrate dev
npm run db:generate
npm run verify
npm run db:status
```

Result:
- Prisma schema valid.
- TypeScript typecheck passed.
- 13 test files passed.
- 59 tests passed.
- production build passed.
- database schema reported up to date with 6 migrations.

Reopen this block only if the session/presence contract changes.
