<!-- Archived source from https://github.com/RagueL-HigaBase/higa_systems_react/blob/main/docs/production/AUTH_ONBOARDING.md; not current canonical state. -->

# Production Block — Onboarding

Status: **DONE**  
Frontend verified: **2026-09-27**

Canonical domain document: `docs/AUTH_AND_SESSION.md`

## Scope
This record closes the React pre-workspace onboarding flow:
- Company Setup after successful login;
- fixed registration intent during setup;
- first-session PIN handoff;
- CREATE pending-activation state;
- JOIN invitation state;
- expired onboarding-token recovery;
- prevention of workspace-shell/profile exposure before workspace access exists.

Backend onboarding has its own verification state and must not be inferred as verified from this frontend record.

## Final frontend contract
- Company Setup uses the registration intent returned by login and does not offer CREATE/JOIN switching.
- The onboarding token remains in component memory only.
- If Company Setup receives `AUTH_REQUIRED` because the onboarding token expired, pending setup state is discarded and the user returns to Sign In for a fresh login/token.
- After setup, normal session/PIN behavior remains backend-authoritative.
- CREATE users without workspace access see only the standalone pending-activation surface.
- JOIN users without workspace access see only the standalone invitation surface.
- Pre-workspace users do not receive the normal workspace navbar, sidebar, footer or Profile Edit surface.
- Once backend session state reports `workspaceAvailable = true`, the normal workspace becomes available.

## Verification
Verified locally by the project owner:

```bash
npm run verify
```

Result:
- TypeScript typecheck passed;
- production Vite build passed.

Non-blocking build warnings:
- third-party `lottie-web` contains `eval`;
- some generated chunks exceed Vite's default size warning threshold.

These warnings do not change the closed Onboarding behavior.

## Reopen only if
Reopen this frontend block if:
- onboarding intent behavior changes;
- the backend onboarding/session contract changes;
- pending activation or invitation routing changes;
- workspace-access rules change.

Otherwise treat the frontend onboarding block as closed.
