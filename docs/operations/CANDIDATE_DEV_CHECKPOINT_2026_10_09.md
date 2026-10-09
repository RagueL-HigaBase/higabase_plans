# Candidate Development Pipeline — end-of-day checkpoint

**Date:** 2026-10-09  
**Status:** Experimental end-to-end pipeline verified by owner; further hardening and reset deletion pending.  
**Repositories:** `higa_systems_express`, `higa_systems_react`, existing `main` only.  
**Authority:** factual checkpoint, **not** approval to implement proposed work or modify canonical Candidate / ESCO architecture.

## What was proven today

1. **Preview:** development-only PDF/DOCX upload (up to 8 MiB), OpenAI parsing and strict `ResumeDraftV1` JSON; Preview does not persist a Candidate.
2. **Experimental save:** `POST /api/dev/candidates/intake` writes Candidate, CandidateContact, ResumeDraft JSONB and CandidateDevIntake relation in one PostgreSQL transaction. The owner saw a saved ID, inspected candidate rows, relation, contact and JSON-based counts directly in pgAdmin; first checked candidate had **8 workHistory records and 22 skills**.
3. **List:** `GET /api/dev/candidates` drives real rows in Operations → Recruitment → Candidates, replacing the previous empty array. Owner observed the saved candidate in the Phoenix table.
4. **Detail:** clicking a candidate's name opens `/recruitment/candidates/:candidateId`, using `GET /api/dev/candidates/:id` to display real identity/contact and linked draft fields. Owner provided screenshots of multiple distinct CV profiles, including short operational and multidisciplinary professional histories. The Control Panel's standalone mock Candidate Prototype was retained.
5. **Reset safety:** `npm run dev:candidates:reset` shows the count of linked experimental candidates and requires explicit `RESET N`. Owner's local command found **4 candidates**; typing `CANCEL` produced `Cancelled. No rows deleted.` **Destructive reset has NOT been exercised or accepted**.

## Local verification evidence

- Owner: Express `npm run verify` successful on 2026-10-09: **35 test files / 150 tests passed**, Prisma validation, TypeScript and build green.
- Owner: `npm run db:status`: **16 migrations**, `Database schema is up to date!`, local `higa_systems` PostgreSQL.
- Owner previously reported React `npm run verify` green for the list/detail pipeline; visual browser screenshots corroborate operational list and candidate details. Do not substitute these for unreported CI or full regression testing.
- Development save, list, and detail are browser-verified. Country intentionally blank where not represented by Candidate Core.
- No new branches, PRs, or unrelated database resets were part of this work.

## Known limitations / bugs (OPEN)

- **CV rate limit:** Preview and Save share **5 requests per 15 minutes**. Browser showed `Too many attempts`; larger 10–15 CV experiments currently hit HTTP 429. Proposed narrow dev-only increase: **30 requests / 15 minutes**; **NOT implemented**.
- **Stale success banner:** on failed subsequent request the previous green Preview may remain next to the red error. Fix should clear stale success state; **NOT implemented**.
- A browser screenshot displayed raw translation key `candidateDevSaved` rather than localized save text; investigate separately if reproducible.
- **Reset deletion** and its effects on other tables have not been manually verified. Do not claim whole reset is DONE or that the database was cleared.
- No deduplication of candidate identity; repeat upload can create duplicate Candidates. No source PDF/DOCX durability, provenance audit, consent workflow or production scoped API.
- `ResumeDraftV1` contains `workHistory[].title`, skills and tools, but no standalone Occupations layer. AI extracts/rewrites items from document; **ESCO classification is not currently invoked** in this CV pipeline. Preserve original source facts separately before any verified normalization. Some personal/employment eligibility facts lack dedicated draft fields and some skill/certificate classifications can differ from intended Higa data domains.
- Current dev list/detail are not Organization-scoped by consent or authorization. **Do not deploy or repurpose for production.**
- AI-BENCHMARK-1 and preexisting unrelated work remain separate; neither is implicitly complete.

## Next working session — recommended order (2026-10-10)

1. **Small approved hardening block (PROPOSED, owner approval required):** raise *development-only* CV Preview+Save rate limit to 30/15min in Express; ensure stale success Preview is cleared on React request error. Add focused tests, update central Express/React task MD, have owner run `git pull` and `npm run verify` in each changed repository and browser check. Do not adjust unrelated auth rate limits, AI provider or production behavior.
2. **Controlled Reset validation (owner decision):** review experimental candidate list/count, optionally execute the explicit `RESET N` in local terminal, then verify Candidate list empty and SystemUser/Organizations/ESCO intact. This is destructive and **must not run automatically**; preserving current samples until reviewed is equally valid.
3. **Batch CV review:** load 10–15 consented/synthetic diverse CVs, inspect structured draft and Candidate Details side by side with originals; catalog extraction omissions, unsupported AI assertions, mistaken certificates/skills and date anomalies before making schema/prompt changes. Avoid needless duplicate saves.
4. **Research discussion, not implementation:** separate AI source extraction, AI interpretation of duties, occupation inference and ESCO `EscoConcept` normalization. WorkHistory is evidence, not automatically a confirmed profession. Decide interface and provenance contract only after sample review.
5. **Defer:** new design, full Candidate permissions/Connections/Organization consent, production ingest, mobile, ESCO matching and major ResumeDraftV2 changes until separately approved.

## Resume instructions for next agent

Read `DOCUMENTATION_INDEX.md`, `AGENT_RULES.md`, `TASKS.md`, `PROJECT_STATE.md`, this checkpoint, relevant `projects/express/CURRENT_TASK.md` and `projects/react/CURRENT_TASK.md`; check live `main` before any mutation. No feature branches/PR. Do not delete candidates merely because reset exists. Do not mark unverified work DONE. Maintain central MD alongside each implementation block.
