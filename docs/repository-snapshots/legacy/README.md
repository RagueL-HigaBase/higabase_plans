<!-- Historical reference copied from https://github.com/RagueL-HigaBase/higa_systems/blob/main/README.md. Not an implementation authority. -->

# Higa Systems

Higa Systems is a TypeScript/Node.js platform with a React web application, PostgreSQL/Prisma persistence, and isolated external-system integrations.

The platform core is intentionally independent from Planbition and other source systems. External systems belong in isolated components and must not become the source of truth for System Core.

## Repository structure

- `apps/api` — HTTP transport, application composition, runtime wiring.
- `apps/web` — React/Phoenix web application.
- `components/system` — System Core authentication, sessions, permissions, profile and password reset logic.
- `components/planbition` — isolated Planbition integration. It is not part of System Core.
- `prisma` — PostgreSQL schema and migrations.
- `tests/integration` — database-backed system scenarios.
- `.github/workflows/ci.yml` — clean CI verification.

## System Core security model

Browser identity is owned by the backend.

- Passwords are stored as scrypt hashes.
- Browser session tokens are opaque random values; only an HMAC digest is persisted.
- A user has one current browser session. A new full login replaces the previous session.
- The 4-digit PIN belongs to the current session, not to the user account.
- A newly created session starts locked and requires PIN setup.
- An unlocked session locks after 15 minutes without authenticated user activity.
- Two incorrect PIN attempts revoke the session.
- Closing the browser tab removes the tab-local unlocked marker, so reopening requires the PIN again.
- Password reset tokens are high-entropy, single-use values. A successful reset revokes the user's current session.
- Viewer access does not include System Admin Panel permission.
- Production browser mutations require same-origin requests and secure cookies.

## Requirements

- Node.js 22.12+ on the 22.x line, or Node.js 24+
- npm
- PostgreSQL 16+ for database-backed development
- Docker with Compose for the containerized stack

## Local development

Create a local environment file from `.env.example` and provide at least:

- `DATABASE_URL`
- `SYSTEM_SESSION_HMAC_SECRET`
- `SYSTEM_OWNER_EMAIL`

Then run:

```bash
npm ci
npm run db:generate
npx prisma migrate deploy
npm run verify
npm run dev
```

The web application can be started separately with:

```bash
npm run web:dev
```

## Verification

The main verification commands are:

```bash
npm run db:validate
npm run typecheck
npm run web:typecheck
npm test
npm run build
npm run web:build
```

`npm run verify` runs the normal local verification chain.

GitHub Actions additionally starts a clean PostgreSQL instance, applies every migration, runs the database-backed System Core HTTP scenario, validates Docker Compose, and builds both the API and web images.

The integration scenario covers the real repository chain for registration, login, session PIN setup, concurrent first profile creation, profile updates, two-attempt PIN revocation, session replacement, password reset single-use behavior, password change, session revocation and logout.

## Docker

The root `Dockerfile` contains three deployment targets:

- `api` — production Node.js API.
- `migrate` — Prisma migration runner.
- `web` — Nginx serving the React build and proxying `/api` to the API service.

`compose.yml` starts PostgreSQL, waits for database health, applies migrations, starts the API only after migrations succeed, and then starts the web container.

Production requires:

- a strong `POSTGRES_PASSWORD`
- a cryptographically random `SYSTEM_SESSION_HMAC_SECRET`
- `SYSTEM_OWNER_EMAIL`
- `PASSWORD_RESET_BASE_URL`
- `PASSWORD_RESET_DELIVERY_URL`
- `PASSWORD_RESET_DELIVERY_TOKEN`

Production password reset delivery uses the HTTP delivery adapter. Local file-based reset delivery is rejected by the production runtime.

The included Nginx service listens on HTTP inside the stack. Put the deployment behind a TLS terminator/reverse proxy before exposing it publicly; production browser authentication uses Secure cookies and HTTPS same-origin protection.

## Health endpoints

- `GET /api/health` — liveness: the API process is running.
- `GET /api/ready` — readiness: the API can reach PostgreSQL.

The API Docker healthcheck uses the readiness endpoint.

## Architecture rules

The engineering contract is documented in `AGENTS.md`. Important rules include:

- keep `apps/api` focused on transport/composition;
- put business logic in components;
- keep external integrations isolated;
- never trust frontend-provided identity;
- keep authentication and authorization separate;
- use `hb_` prefixes for Higa-owned database fields and integration-specific prefixes for external data;
- keep secrets, generated session state and runtime data out of Git;
- make changes in small verified blocks.

## Planbition

Planbition remains an isolated external integration under `components/planbition`. The current System Core hardening work intentionally does not change Planbition behavior.
