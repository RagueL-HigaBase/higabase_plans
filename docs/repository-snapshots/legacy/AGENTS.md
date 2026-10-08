<!-- Historical reference copied from https://github.com/RagueL-HigaBase/higa_systems/blob/main/AGENTS.md. Not an implementation authority. -->

# Higa Systems Engineering Rules

These rules are mandatory for future Codex work in this repository.

## 1. Development approach

- Work in small, reviewable, verified blocks.
- Do not implement unrelated future functionality.
- Do not refactor neighboring modules unless the current task requires it.
- After each block, run the relevant typecheck/tests and show the exact diff.
- Prefer simple explicit code over hidden magic.

## 2. Separation of responsibilities

- One file/module should have one clear responsibility.
- Do not create large files containing unrelated logic.
- Separate configuration, domain logic, persistence, transport, validation, crypto, middleware, and external integrations.
- Do not create artificial files/types/classes when they are not genuinely needed.
- Keep dependencies flowing in a clear direction.
- Domain modules should own their domain-specific configuration and logic.

## 3. Domain boundaries and naming

Use explicit domain ownership:

- `System*` = Higa Core / internal system.
- `Planbition*` = Planbition integration.
- `Easyflex*` = EasyFlex integration.
- Other external systems must follow the same pattern.

External integrations must not define Higa Core architecture.

SQL table examples:

- `system_users`
- `system_sessions`
- `planbition_work_hours`

## 4. Database rules

- Higa Core database fields use the `hb_` prefix.
- External integrations retain their own database field prefixes (`pb_` for Planbition, and integration-specific prefixes for other systems).
- These prefixes apply to persisted columns; Prisma relation-only fields are not database columns.
- Prisma field names must match PostgreSQL column names exactly.
- Do not use field-level `@map` aliases.
- Model-level `@@map` is allowed for SQL table naming.
- Prefer explicit relations and referential behavior.
- Do not introduce cascade delete unless explicitly agreed.
- External integration identifiers are metadata and must not automatically become Higa Core identifiers.
- Do not create unnecessary foreign keys between independent integration domains.

## 5. Authentication and session security

- Browser authentication uses one opaque System session token.
- The raw session token must never be stored in the database.
- Store only the deterministic server-side token hash/HMAC.
- The raw session token will eventually live only in a secure httpOnly cookie.
- Frontend JavaScript must not read the session token.
- Never expose password hashes, session hashes, HMAC secrets, or other credentials to the frontend.
- Never log credentials or secrets.
- No hardcoded secrets or fallback secrets.

## 6. Server-side identity

- Authentication identity must always be derived server-side from the validated `SystemSession`.
- The backend resolves: session token -> token hash -> `SystemSession` -> `hb_user_id`.
- Frontend must never choose or submit `hb_user_id` (or any other user ID) for authorization decisions.
- Never trust a frontend-provided user ID as proof of identity.
- Internal IDs may be returned only when they are genuinely application data, never as authentication credentials.

## 7. Authorization

- Authentication and authorization are separate responsibilities.
- Session validation determines who the user is.
- Permissions/memberships determine what the authenticated user may access.
- Do not mix permission logic into session token crypto utilities.

## 8. Session inactivity / PIN design

Preserve these architectural requirements for future implementation:

- User chooses a temporary 4-digit PIN during full login.
- PIN is specific to that login/session, not a permanent account PIN.
- Real authenticated API activity refreshes the PIN idle window.
- Passive open browser tabs/heartbeats must not count as user activity.
- After the configured inactivity period, the main session may remain valid but protected actions require PIN verification.
- Correct PIN resumes the existing session.
- Incorrect PIN revokes the session and requires full login again.
- PIN must only be stored as a secure hash.
- Keep main session expiration separate from PIN inactivity expiration.

Do not implement PIN logic unless it is explicitly part of the requested task.

## 9. External integrations

- Planbition and other integrations must remain isolated from System Core.
- Do not couple System auth/domain models to Planbition models unless explicitly required.
- Do not modify integration code as a side effect of Core work.

## 10. Frontend boundary

- Frontend is a client of the API, not a source of authentication truth.
- Do not expose backend credentials or security internals to React.
- Prefer endpoints such as `/api/me` where server identity comes from the session.
- Authorization decisions must remain on the server.

## 11. Quality gates

Before finishing a code task, when applicable:

- Run `npm run typecheck`.
- Run `npm test`.
- Run Prisma validation when the Prisma schema changes.
- Show the exact diff.
- Stop after the requested scope.

## 12. Future Public API

- Higa must support a versioned Public API for external developers in the future.
- Public API requirements must not alter or weaken the browser authentication security model.
- The Public API must never expose password hashes, session hashes, HMAC secrets, or other secrets.
- Do not implement the Public API or its authentication unless explicitly requested.

## Repository architecture and component ownership

Higa Systems uses strict physical component boundaries.

### apps/

`apps/` contains runnable applications and transport/composition layers only.

Examples:

- `apps/api` = HTTP API application
- `apps/web` = React frontend application

`apps/api` may contain:

- HTTP routes
- controllers
- request/response validation
- middleware
- API bootstrap
- application composition
- dependency wiring
- API plugins/adapters for components

`apps/api` must NOT contain:

- external integration implementation
- component business/domain logic
- reusable component services
- component-specific parsers
- file/XLSX/CSV processing logic
- image processing logic
- external API clients
- component persistence logic

### components/

Every independent subsystem, external integration, or functional component must live under `components/`.

Examples:

- `components/system`
- `components/planbition`
- `components/easyflex`
- `components/workforce`
- `components/recruit-robin`

All logic that belongs to a component must remain inside that component's folder.

Examples:

- Planbition session/login/report/export/parsing logic -> `components/planbition`
- System authentication/users/permissions logic -> `components/system`
- EasyFlex integration logic -> `components/easyflex`

A component may contain its own:

- configuration
- services
- clients
- parsers
- validation
- persistence adapters
- errors
- types
- internal utilities
- tests

Components must not depend on Express or API transport code.

Components must be reusable from:

- API
- CLI
- workers
- scheduled jobs
- future applications

### Internal component structure

Inside a component, keep responsibilities physically separated.

Do not combine unrelated responsibilities into one large file.

Prefer dedicated folders/files for:

- configuration
- domain types
- validation
- services
- repositories/persistence
- authentication/session logic
- transport adapters
- parsers
- errors
- utilities
- tests

A file should have one primary responsibility.

If a file starts handling multiple concerns
(validation + persistence + orchestration + parsing + transport),
split it before adding more behavior.

Do not create generic "helpers.ts", "utils.ts", "service.ts", or "manager.ts"
files that become catch-all containers.

Keep dependency direction explicit and predictable.
Higher-level orchestration may depend on lower-level modules,
but lower-level modules must not depend back on orchestration or transport.

### API component plugins

When `apps/api` exposes or manages a component, all API-specific code for that component must live in a dedicated API plugin folder.

Required structure pattern:

```text
apps/api/src/plugins/
  planbition/
  system/
  easyflex/
  ...
```

Example dependency direction:

```text
apps/api/src/plugins/planbition
-> components/planbition
```

Rules:

- API plugin code may depend on its component.
- Components must never depend on API plugins.
- Express-specific code belongs only in the API/plugin layer.
- Routes/controllers/request validation for one component must stay inside that component's API plugin folder.
- Do not mix API code for different components in one generic folder.
- Do not put component business logic into API plugins.

### Ownership rule

Before creating or moving a file, determine which component owns the logic.

If logic belongs to a component, it must live in that component.
If code only exposes that component through HTTP, it belongs in the API plugin for that component.

Do not place code in `apps/api` merely because the API currently calls it.

### Shared code

Use `packages/shared` only for genuinely reusable technical code used by multiple components/applications.

Do not move code into shared merely because two files look similar.
Prefer component ownership unless true cross-component reuse exists.

- Do not create empty component folders for future integrations.
