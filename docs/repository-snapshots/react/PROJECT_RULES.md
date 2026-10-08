<!-- Historical main-branch documentation snapshot; not live authority. Original: https://github.com/RagueL-HigaBase/higa_systems_react/blob/main/PROJECT_RULES.md. For current shared state consult central PROJECT_STATE.md. -->

# Higa Systems React — Project Rules

Last reviewed: 2026-10-02

Read this file before changing frontend behavior.

## Git workflow — owner-mandated single branch

**Mandatory, permanent rule:** work exclusively on the existing `main` branch. NEVER create a feature/task/temporary branch, NEVER switch branches (locally or remotely), NEVER open a PR as a substitute for direct `main` commits, and NEVER instruct the owner to checkout a feature branch. Read-only inspection of historical commits is permitted for recovery, but it must never mutate or check out another branch.

Before writing, inspect current `main` and its active task. Make small scoped commits directly to `main`; when reverting regressions, preserve unrelated work and use targeted inverse changes rather than destructive resets. Do not run `git reset --hard`, `git clean`, automatic `git stash pop`, or rewrite Git history. If a direct `main` update is blocked, stop and obtain explicit owner approval rather than creating a branch. Keep both local repositories on `main` and use `git fetch origin` followed by `git merge --ff-only origin/main` after checking worktree status.

## 1. Source-of-truth order
Use project documentation in this order:
1. `PROJECT_RULES.md` — global frontend constraints;
2. `CURRENT_TASK.md` — temporary active implementation scope when its status is not `NO ACTIVE TASK`; it may refine the next change but never override PROJECT_RULES or pretend unfinished work is implemented;
3. `PROJECT_STATE.md` — current implemented state and known gaps;
4. domain documents under `docs/`.

Domain documents:
- `docs/AUTH_AND_SESSION.md`;
- `docs/SYSTEM_USER_PROFILE.md`;
- `docs/ORGANIZATION.md`;
- `docs/SYSTEM_ADMIN.md`;
- `docs/PHOENIX_UI.md`.

Closed verified production blocks live under `docs/production/`. These records stay concise and complement, rather than replace, canonical domain documents.

Future/planned design notes live under `docs/planning/`.
They describe intended work only and must never be treated as implemented state or as stronger authority than PROJECT_RULES, CURRENT_TASK, PROJECT_STATE, or canonical domain docs.

Do not duplicate long domain specifications back into PROJECT_STATE.md.

## 2. Repository authority
Active frontend:
`RagueL-HigaBase/higa_systems_react`

Backend:
`RagueL-HigaBase/higa_systems_express`

Phoenix reference:
`RagueL-HigaBase/phoenix-react-reference`

Phoenix is read-only and is the structural/visual authority for approved Phoenix-derived UI.

Detailed Phoenix rules live in `docs/PHOENIX_UI.md`.

Higa information typography (A1/A2) is a local semantic overlay implemented through reusable components, never through global Phoenix CSS overrides. Its canonical rules and component mapping live in `docs/PHOENIX_UI.md`.

## 3. Business-only scope
The active web client is organization/business oriented.

Candidate/job-seeker/mobile-specific UI is not part of the active product unless a new explicit architecture decision is made.

The authenticated business SystemUser profile and organization-scoped Location Profile are valid business surfaces and must not be confused with removed candidate profile code. The former standalone Organization Company Profile is obsolete.

## 4. Change discipline
- The project owner selects scope.
- Work in small approved blocks.
- Do not introduce unrelated future modules or speculative refactors.
- Preserve approved Phoenix structure when adapting business content.
- Do not add custom CSS or replacement visual systems without explicit approval.
- Permanent constraints belong here.
- Implemented/current facts belong in PROJECT_STATE.md or the relevant domain document.
- Do not keep historical development narrative in current-state documentation.
- Before implementation, read `CURRENT_TASK.md`.
- `CURRENT_TASK.md` is temporary working state: unfinished decisions stay there and are not promoted to PROJECT_STATE or domain docs.
- On task completion, reconcile durable facts into PROJECT_STATE/domain docs as appropriate, add/update the relevant `docs/production/*` record after verification, then reset `CURRENT_TASK.md` to `NO ACTIVE TASK`.
- Implementation work should start only from an explicitly approved task scope; recommended task statuses are `DISCUSSION`, `READY`, `IN PROGRESS`, `BLOCKED`, `VERIFY`, and `DONE`.
- Every change is committed clearly.
- Do not claim verification passed until the project owner reports it.

## 5. Phoenix zero-invention rule
For every Phoenix-derived UI change:
- inspect the matching Phoenix source immediately before implementation;
- preserve original structure, classes, Bootstrap/Phoenix patterns and behavior;
- adapt only approved business content, data, routing, validation and i18n unless structural deviation is explicitly approved;
- do not invent CSS, wrappers, spacing systems or substitute components.

The complete rule is in `docs/PHOENIX_UI.md`.

## 5a. Phoenix form-pattern vocabulary
For Higa Systems forms, `src/pages/apps/events/CreateAnEvent.tsx` in the Phoenix reference defines the two approved form families.

- **Primary Form Pattern** = the left/content column of Phoenix Create an Event. Use it for principal data-entry forms and core entity data.
- **Secondary Form Pattern** = the right/settings column of Phoenix Create an Event. Use it for auxiliary settings, compact configuration, option groups and supporting controls.

When starting a new form, identify each block as Primary or Secondary before implementation. If the intended family is not clear from the approved screen or current task, ask the project owner before writing the UI.

Do not create a third form family, hybrid spacing system, custom form CSS or substitute control structure unless the project owner explicitly approves it.

Detailed source mappings and spacing rules are in `docs/PHOENIX_UI.md`.

## 6. Backend authority
Authentication, membership, roles, workspace availability and security decisions are backend-authoritative.

Frontend must not invent:
- organization membership;
- organization role;
- platform role;
- SYSTEM_OWNER status;
- workspace access.

## 7. Internationalization
All visible product UI added or adapted for Higa Systems must use the existing 28-language business locale system.

Do not hardcode user-visible business labels when an i18n key should exist.

Country/date localization should use the active language and existing project patterns.

## 8. Business-text invariant
Persisted user-entered business text and identifiers are ASCII-only.

Frontend should filter unsupported non-ASCII input where practical and validate through Zod where the form has a schema.

Backend validation remains authoritative.

Passwords and opaque tokens are exempt.

## 9. Organization boundary
Organization editing must use the explicit organization-scoped backend contract.

Do not invent a frontend-only global "current company" assumption that bypasses organizationId/RBAC.

See `docs/ORGANIZATION.md`.

## 10. Storage
Do not implement local-filesystem media persistence.

Current image upload controls remain preview/UI-only until approved production object storage is connected.

## 11. Verification
Standard frontend verification:

```bash
npm run verify
```

Run `npm install` when dependencies or lockfile changed.

Do not mark verification complete until the project owner reports the local commands green.
