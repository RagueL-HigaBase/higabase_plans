# AGENT_RULES — mandatory operating contract

Effective 2026-10-08. Applies to agents working on all HigaBase repositories. Read before any code or documentation mutation.

## 1. Git: main only
- Work exclusively on the existing `main` branch. No feature, task, temporary or recovery branches; no pull requests unless the owner explicitly asks.
- Never switch away from `main` for implementation. Read-only historical inspection is allowed.
- Do not merge, cherry-pick, delete branches, force push, reset --hard, or clean worktrees without an explicit task and verified preservation of unique changes.
- Legacy branches remain untouched until Candidate + ESCO closure and a separate owner-approved cleanup.

## 2. Strict scope: do only what is requested
- Implement precisely the requested change. Example: a button request authorizes a button only, not an API, table, new flow, permission system or architecture.
- No speculative features, dependencies, visual restyling, refactors, migrations, new abstractions, pages, tests outside necessary verification, or documentation rewrites unrelated to the task.
- Read related code/docs to avoid damage, but do not expand the delivery scope.
- If fulfilling the request *requires* additional behavior or architecture, describe the dependency and get explicit owner approval **before** implementing it.
- Proposals are proposals, not permission to execute. Never interpret prior discussion, planning notes or a suggested roadmap as approval of the next block.

## 3. Architecture change gate
- The owner is the decision authority. Identify the existing canonical contract, state the concrete proposed change, affected domains/files, migration/data consequences, security implications, alternatives and verification plan.
- Obtain explicit approval of the scope **before** changing architecture, database models/migrations, cross-service contracts, identity, security, authorization, consent, or ownership boundaries.
- Do not silently reinterpret central rules. If implementation conflicts with architecture, mark BLOCKED and ask for the decision.

## 4. Mandatory agent startup
1. Read central `DOCUMENTATION_INDEX.md`, `AGENT_RULES.md`, `PROJECT_STATE.md` and `docs/operations/NEXT_STEPS.md`.
2. Read the applicable canonical architecture completely and the active repository's `main` documents/code.
3. Check `git status`, branch and relevant source/test reality; never overwrite dirty local changes.
4. Confirm only the requested scope, identify affected files, avoid unrelated work.
5. Work in minimal verifiable blocks; record exact tests, state and what was not checked.

## 5. State and quality gates
- Label work: PROPOSED, READY, IN PROGRESS, VERIFY, DONE, BLOCKED. Never mark DONE without necessary tests and owner checks.
- Green unit tests do not imply database migration parity, correct production behavior or visual acceptance.
- Keep distinct: intended architecture, code implemented, actual local DB state, CI, owner visual verification and open regressions.
- Regressions suspected after prior merges must be audited, not assumed fixed; never resurrect entire old branches blindly.
- When a block closes, update canonical state and related documentation **only** as required for that block.
- Report remaining gaps and do not quietly change them.

## 6. Current owner priority
1. Finish approved Candidate + ESCO blocks.
2. Perform independent functional/regression audit starting from Sign In and navigation, compare expected vs actual behavior.
3. Fix confirmed defects in small approved blocks.
4. Run SonarQube remediation and close each block with evidence.
5. After Candidate + ESCO, inspect and clean historical branches with separate approval.

Nothing in this document approves implementing future roadmap items automatically.
