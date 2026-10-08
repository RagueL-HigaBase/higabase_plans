# Higa Systems React

Business-only React client for Higa Systems.

## Stack
- React 19
- TypeScript
- Vite
- React Bootstrap / Bootstrap
- Zod
- Phoenix React reference patterns

## Current scope
- business account registration/sign-in;
- first-login Company Setup;
- session PIN setup/unlock;
- pending organization activation;
- invitation acceptance;
- protected Phoenix workspace shell;
- SystemUser Edit Profile;
- Organization Dashboard and administration shell;
- reusable persisted Location shell and Location Profile;
- Location Team frontend template;
- SYSTEM_OWNER organization activation queue.

Candidate/job-seeker/mobile-specific UI is outside the active web client.

## Development

```bash
npm install
npm run dev
```

Vite proxies `/api` to the backend development server.

## Verification

```bash
npm run verify
```

## Phoenix
Phoenix is a read-only reference and the source of truth for approved UI structure.

Read `docs/PHOENIX_UI.md` before modifying any Phoenix-derived component or page.

## Documentation
Read in this order:
1. `PROJECT_RULES.md`;
2. `CURRENT_TASK.md`;
3. `PROJECT_STATE.md`;
4. the relevant file in `docs/`.

Domain documentation:
- `docs/AUTH_AND_SESSION.md`
- `docs/SYSTEM_USER_PROFILE.md`
- `docs/ORGANIZATION.md`
- `docs/SYSTEM_ADMIN.md`
- `docs/PHOENIX_UI.md`

Backend repository:
`higa_systems_express`.
