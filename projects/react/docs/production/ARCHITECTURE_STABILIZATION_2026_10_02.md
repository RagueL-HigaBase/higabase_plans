# Frontend Architecture Stabilization — 2026-10-02

Verified: 2026-10-02

## Scope
Closed the frontend architecture pass for Member Details, Location Details, session availability UI, Higa information typography, and removal of superseded experimental code.

## Final contract
- Phoenix remains the global layout/theme/grid authority.
- Higa A1/A2 information typography is a local reusable overlay through `HigaInfoSurface`.
- Member Details uses persistent member context plus a right-side workspace.
- Location Details uses top tabs as the full workspace switch; Overview content belongs inside the Overview tab.
- The signed-in user's Profile view is the organization Member Details page.
- `/profile/edit` remains the SystemUser editor and returns to Member Details after Save.
- Presence and availability are separate: Online/Offline is system-derived; Available/Busy/Away is selectable and persisted.
- Obsolete standalone Profile page, Location Helper experiment, Anna Carry mock data, and Member typography-lab code are removed.

## Verification
Project owner verified locally:

```bash
npm run verify
```

Result:
- TypeScript typecheck passed.
- Vite production build passed.

Known non-blocking warnings remain:
- third-party `lottie-web` uses `eval`;
- generated bundle chunks exceed Vite's default warning threshold.

Reopen this block only for a deliberate architecture change, not for normal feature work.
