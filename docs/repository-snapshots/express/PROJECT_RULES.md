<!-- Historical source snapshot from higa_systems_express/main. This is evidence, not canonical current state. For live state see PROJECT_STATE.md in HigaBase Plans. Original: https://github.com/RagueL-HigaBase/higa_systems_express/blob/main/PROJECT_RULES.md -->

# Higa Systems Express — Project Rules

Last reviewed: 2026-10-04

Read this file before changing backend behavior.

## Git workflow — owner-mandated single branch

**Mandatory, permanent rule:** work exclusively on the existing `main` branch. NEVER create a feature/task/temporary branch, NEVER switch branches (locally or remotely), NEVER open a PR as a substitute for direct `main` commits, and NEVER instruct the owner to checkout a feature branch. Read-only inspection of historical commits is permitted for recovery, but it must never mutate or check out another branch.

Before writing, inspect current `main` and its active task. Make small scoped commits directly to `main`; when reverting regressions, preserve unrelated work and use targeted inverse changes rather than destructive resets. Do not run `git reset --hard`, `git clean`, automatic `git stash pop`, or rewrite Git history. If a direct `main` update is blocked, stop and obtain explicit owner approval rather than creating a branch. Keep both local repositories on `main` and use `git fetch origin` followed by `git merge --ff-only origin/main` after checking worktree status.

## 1. Source-of-truth order
Cross-project HigaBase invariants are owned by:
- [HigaBase System Architecture](https://github.com/RagueL-HigaBase/HigaBase_Plans/blob/main/SYSTEM_ARCHITECTURE.md)

Use documentation in this order:
1. central `HigaBase_Plans/SYSTEM_ARCHITECTURE.md` — cross-project identity, organization/location, invitation, audit and permission-boundary invariants;
2. `PROJECT_RULES.md` — backend-specific constraints that must not contradict central architecture;
3. `CURRENT_TASK.md` — temporary active implementation scope when its status is not `NO ACTIVE TASK`; it may refine the next change but never override central architecture or PROJECT_RULES;
4. `PROJECT_STATE.md` — current implemented state and known gaps;
5. domain documents under `docs/`.

Domain documents:
- `docs/AUTH_AND_SESSION.md`;
- `docs/SYSTEM_USER_PROFILE.md`;
- `docs/ORGANIZATION.md`;
- `docs/SYSTEM_ADMIN.md`;
- `docs/DATABASE.md`;
- `docs/ESCO.md`.

Closed production blocks live under `docs/production/`. These files record the final verified contract for a completed block and must stay concise. They do not replace the canonical domain documents; they point back to them and capture only the decisions, verification status and reopen conditions for that closed block.

Do not duplicate long domain specifications back into PROJECT_STATE.md or production records.

## 2. Scope and stack
This repository is the business web backend/API only:
- Node.js;
- Express 5;
- TypeScript;
- Zod;
- Prisma 7;
- PostgreSQL;
- Vitest.

The owner approved an **isolated Candidate Core schema-only foundation** on 2026-10-08, following `HigaBase_Plans/CANDIDATE_ARCHITECTURE.md` §20. This is a narrow exception to the original business-only scope: `Candidate` and `CandidateContact` PostgreSQL models/migration only. No Candidate API, candidate login, processing/consent or mobile business logic is authorized by this exception. All subsequent Candidate domain behavior needs separate approval.

## 3. Change discipline
- The project owner selects scope.
- Work in small approved blocks.
- Do not introduce unrelated future modules or speculative refactors.
- Behavior changes require tests.
- Permanent constraints belong here.
- Implemented/current facts belong in PROJECT_STATE.md or the relevant domain document.
- Do not keep historical development narrative in current-state documentation.
- Before implementation, read `CURRENT_TASK.md`.
- `CURRENT_TASK.md` is temporary working state: unfinished decisions stay there and are not promoted to PROJECT_STATE or domain docs.
- On task completion, reconcile durable facts into PROJECT_STATE/domain docs as appropriate, add or update the relevant `docs/production/*` record for a verified closed block, then reset `CURRENT_TASK.md` to `NO ACTIVE TASK`.
- Implementation work should start only from an explicitly approved task scope; recommended task statuses are `DISCUSSION`, `READY`, `IN PROGRESS`, `BLOCKED`, `VERIFY`, and `DONE`.
- Every change is committed clearly.
- Do not use `npm audit fix --force` automatically.
- Do not claim local verification passed until the project owner reports it.

## 4. Naming and database ownership
All Higa-owned PostgreSQL objects use the `hb_` prefix.

Prisma business/system models use readable `System...` names.

The independent Candidate domain uses Prisma `Candidate...` models and physical `hb_candidate...` / `hb_candidates` tables. Candidate names/source data permit multilingual Unicode; business SystemUser ASCII validation must not be copied to Candidate.

The independent ESCO reference domain uses the `Esco...` prefix for Prisma models and `hb_esco_...` for physical PostgreSQL objects.

Detailed migration rules live in `docs/DATABASE.md`.

## 5. Business-text invariant
Persisted user-entered business text and identifiers are ASCII-only.

Allowed:
- English/Latin ASCII letters;
- digits;
- field-appropriate printable ASCII punctuation.

Rejected:
- Cyrillic;
- accented Latin letters;
- other non-ASCII Unicode.

Backend Zod validation is authoritative.
Frontend should mirror the rule.

Passwords, opaque tokens and cryptographic material are exempt.

Person names additionally follow the stricter English/ASCII passport-name rule described in `docs/SYSTEM_USER_PROFILE.md`.

## 6. Authentication and security
Registration never grants privileged roles.

Backend session, onboarding, password-reset and PIN invariants are defined in `docs/AUTH_AND_SESSION.md`.

Security-sensitive access decisions are always backend-authoritative.

## 7. RBAC
Organization roles:
- USER;
- ADMIN;
- OWNER.

Platform roles:
- VIEWER;
- ADMIN;
- SYSTEM_OWNER.

Organization and platform scopes are separate.

There may be only one SYSTEM_OWNER.

Public registration never grants privileged roles.

Organization profile editing requires ADMIN or OWNER membership for the target organization.

## 8. Organization boundary
The central architecture defines the business invariant:

```text
1 SystemUser
→ maximum 1 business Organization
→ zero or more Locations inside that Organization
```

Organization APIs must remain organization-id scoped when the operation belongs to a specific organization.

Do not use the one-Organization invariant as a reason to weaken backend scope validation. Explicit target Organization/Location ownership checks remain required.

Organization rules and profile persistence live in `docs/ORGANIZATION.md`.

## 9. Error contract
Known API errors use:

```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "..."
  }
}
```

Frontend behavior branches on stable error codes, not message text.

## 10. Storage
Production media fields store object-storage keys only.

Do not add local-filesystem persistence for profile/company images.

## 11. Migration discipline
The clean business baseline must remain intact.
Future schema changes append normal Prisma migrations.

Never restore the removed historical candidate/mobile migration chain.

See `docs/DATABASE.md`.

## 12. Verification
Standard backend verification:

```bash
npm run db:generate
npm run verify
npm run db:status
```

When schema/migrations change, run the appropriate Prisma migration command first.

Do not mark verification complete until the project owner reports the local commands green.
