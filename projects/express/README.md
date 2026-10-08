# Higa Systems Express

Business-only backend/API for Higa Systems.

## Stack
- Node.js / Express 5
- TypeScript
- Zod
- Prisma 7
- PostgreSQL
- Vitest

## Current scope
- business account registration/authentication;
- password reset;
- session/PIN security;
- first-login company onboarding;
- SystemUser profile;
- organization creation/activation/invitations;
- organization and reusable Location Profile persistence;
- organization/platform RBAC;
- permanent SYSTEM_OWNER bootstrap.

Production Candidate identity, Resume, consent and mobile domains are not part of the current business API contract. Isolated experimental ResumeDraft parsing/storage and AI benchmarks exist for research; they are not a production Candidate API. See docs/planning/CANDIDATE_PROCESSING_AUDIT_2026_10_08.md.

## Development

```bash
npm install
npm run dev
```

## Verification

```bash
npm run db:generate
npm run verify
npm run db:status
```

When a Prisma schema change introduces a migration, run the appropriate migration command before verification.

## Documentation
Read in this order:
1. cross-project `HigaBase_Plans/SYSTEM_ARCHITECTURE.md`;
2. `PROJECT_RULES.md`;
3. `CURRENT_TASK.md`;
4. `PROJECT_STATE.md`;
5. the relevant file in `docs/`.

Domain documentation:
- `docs/AUTH_AND_SESSION.md`
- `docs/SYSTEM_USER_PROFILE.md`
- `docs/ORGANIZATION.md`
- `docs/SYSTEM_ADMIN.md`
- `docs/DATABASE.md`
- `docs/AI_CORE.md`
- `docs/ESCO.md`
- `docs/planning/CANDIDATE_PROCESSING_AUDIT_2026_10_08.md`

The frontend is maintained separately in `higa_systems_react`.
