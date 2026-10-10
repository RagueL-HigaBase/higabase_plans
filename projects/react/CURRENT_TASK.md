# Higa Systems React — Current Task

Status: **VERIFY**

## Recruitment Candidates table — Phoenix CRM Leads scaffold

Owner-approved scope:
- reuse the original Phoenix CRM Leads table structure, already adapted for System Organizations;
- show the same table mechanics in `/recruitment/candidates`;
- add the primary **Add Candidate** button;
- preserve the existing compact page header and navigation;
- do not import Phoenix demo leads or list System Organizations as candidates;
- do not implement Candidate API, storage, forms, ESCO, screening, or Candidate Prototype changes.

Implementation:
- empty candidate table schema: name, email, phone, country, created date;
- search, sorting, selection, pagination and date filter wired to the table state;
- Add Candidate opens a clearly informational modal, not a working create form;
- advanced filters remain disabled until a candidate filtering contract exists.

UI references inspected:
- `phoenix-react-reference/src/pages/apps/crm/Leads.tsx`;
- `phoenix-react-reference/src/components/tables/LeadsTable.tsx`;
- `src/pages/system/Organizations.tsx`;
- `src/components/tables/OrganizationsTable.tsx`;
- `src/pages/organization/OrganizationLocation.tsx`.

Verification required:
- owner runs `npm run verify` locally on updated `main`;
- owner verifies visual alignment and table empty state.
- Candidate listing and creation backend are **not implemented**.

Do not close this block before owner verification.

## CV-UPLOAD-PREVIEW-1 — approved small integration (2026-10-08)

Status: **VERIFY** (local owner check pending).

The existing `Add Candidate` button now opens a Phoenix `Dropzone` for one PDF/DOCX file (8 MiB maximum), then calls a dedicated development-only authenticated backend preview route:
`POST /api/dev/resume-drafts/preview-file`.

- Frontend route: `/recruitment/candidates`.
- API contract: `src/api/resume-preview.ts`; send file bytes directly with matching Content-Type and URL-encoded filename header.
- Dropzone remains Phoenix-derived, without new CSS or dependencies; controlled remove/select notifications are additive.
- Modal shows processing/error state and limited parsed identity/contact preview.
- Results are **not** saved as Candidates, Resumes or ResumeDrafts.
- Preview action exists only in Vite development; production UI does not offer an unimplemented backend upload.
- UI copies use 28 language packs.
- OpenAI file input only; local Ollama still accepts plain text, not a binary CV.

No Candidate Core, consent/relation logic, ESCO mapping, Candidate listing API, Candidate Prototype or advanced permissions were changed.

Owner verification required:
1. Pull latest `main` for backend and frontend.
2. Backend: `npm run db:generate && npm run verify && npm run dev` with `AI_PROVIDER=openai` and `OPENAI_API_KEY`.
3. Frontend: `npm run verify && npm run dev`.
4. Sign in to an unlocked Organization workspace, open Candidates → Add Candidate; test PDF/DOCX, invalid file, limit, no-provider message and absence of new Candidate rows.
5. Confirm no CVs or personal data were committed or persisted by this preview.

Keep the block VERIFY until the owner reports the local result.

## WORKSPACE-RECOVERY-2026-10-08 — owner-requested urgent restoration

Status: **VERIFY** — GitHub CI required and owner visual verification pending.

Root cause: the owner's previous local commit `35456e7` contained Candidate Dossier, Verify Email, masked PIN UI and split Organization/Locations navigation which were absent from the main baseline. A switch to main hid these features; this was not caused by the CV preview code itself. See `docs/planning/WORKSPACE_RECOVERY_2026_10_08.md`.

Restored directly in `main`, without creating or switching branches:
- PIN masked fields and previous UX;
- Sign In missing-verification-email link and full email verification/profile completion bridge;
- System Candidate Prototype and original ECharts chart dependencies;
- Organization Dashboard / Departments / Team and separate Locations Main Location navigation;
- corresponding routes and localized labels from the previously working UI.

Preserved: global Dashboard; Recruitment/Candidates table; Add Candidate experimental CV preview; current backend schemas and ESCO. Do not close until owner has locally checked main and visually confirmed PIN, email recovery, Organization/Locations and Candidate Prototype.

## PIN-SUBMIT-ALIGNMENT — 2026-10-09

Status: **VERIFY — owner local browser and build check pending**.

Owner requested a narrow PIN setup/unlock button alignment fix: while submitting, shared `Button` applies `d-flex`, shifting the button label left. On React `main` commit `a72b59c923d68318e370d4c5a0dcadcb54802a1b`, `src/components/modules/auth/PinForm.tsx` adds Bootstrap `justify-content-center` to the existing full-width PIN submit button. No shared Button implementation, PIN flow, API, state handling, backend, or styling files were changed.

Verification outstanding: owner pulls React `main`, runs `npm run verify`, and checks centered labels while idle and submitting for both Create PIN and Unlock PIN. This has not yet been confirmed in the owner's environment; do not mark DONE.

## PIN-BROWSER-PASSWORD-MANAGER — 2026-10-09

Status: **VERIFY — owner browser check pending**.

Owner reported Google Password Manager leaked-password warning on four-digit session PIN. React `main` commit `af9d51c98052f3cfb568a71741005d6912ea8b56` changed only `src/components/modules/auth/PinForm.tsx`: four PIN controls use `type="text"` and `autoComplete="off"` rather than `type="password"` / `one-time-code`; existing Chrome/WebKit `WebkitTextSecurity: 'disc'` retains dot masking. No backend, API, PIN validation, submit flow or shared Button changes.

Check locally: `npm run verify`; confirm PIN masking, keyboard input/focus, setup and unlock, and that Chrome Password Manager no longer shows a breached-password warning. Browser behavior is not guaranteed across all engines; if a non-WebKit browser reveals digits, report and revisit masking separately with approval. Do not mark DONE before owner confirmation.

## LANGUAGE-PARITY-1 — shared 28-locale auth.json namespace (2026-10-09)

Status: **VERIFY — owner local check and rendered/semantic review pending**.

React `main` received scoped translation completeness changes in `public/locales/*/auth.json`. The shared namespace includes Authentication, Location and Team invitation UI; all 28 locale files now parse as JSON and match the English 400-key baseline exactly (no missing/extra/blank string values; placeholder variable sets match). Existing strings were retained, and absent localized Location and invitation messages were added. The first source-level completeness check is complete, **not** full UI/native-language acceptance.

Verification pending: owner `git pull --ff-only origin main`, `npm run verify`, browser inspection across 28 languages and native/semantic translation review as required. No backend/API/schema modifications. Detailed current audit: `higabase_states/pages/auth/TRANSLATIONS_AUDIT.md`. Do not mark DONE prematurely.

## OPERATIONS-NAV-SKELETON — 2026-10-09

Status: **VERIFY — owner local build and UI check pending**.

Owner-approved navigation-only change on React main commit `8b68732dc7183f0191ef4a8cf1dda6f791c3da36`.

- One visible top-level Dashboard link `/dashboard`.
- Immediately below Dashboard, new `Operations` section with three expandable groups:
  - Recruitment: Candidates (existing `/recruitment/candidates`), Vacancies, Requests, Placements.
  - Transport: Fleet, Drivers, Routes, Schedules.
  - Housing: Properties, Residents, Occupancy, Maintenance.
- Added eleven protected route definitions backed by minimal heading-only placeholder for sections not yet implemented. This does NOT implement backend APIs or module business logic.
- Old Components/Recruitment, Organization, Locations, and System sections remain below Operations. The existing Organization Dashboard **route** `/organization/dashboard` remains available, but its duplicate sidebar menu item is hidden to leave one visible Dashboard.
- Existing Recruitment Candidates page unchanged. Legacy Recruitment sidebar link intentionally retained and therefore may repeat the Candidates destination until a separate cleanup decision.
- No new permissions, schema, API endpoints, role or existing page behavior changes.
- New menu labels are currently literal English strings; **28-language navigation localization still pending** and should not be claimed completed. No translation files changed.

Owner update from React folder:
```powershell
git status -sb
git pull
npm run verify
npm run dev
```
Check only one Dashboard, Operations order and expansion, all 12 operational submenu links (Candidates + 11 placeholders), and unchanged old menu below. Do not mark DONE until owner verifies.

## APP-ADMIN-NAV-SPLIT — 2026-10-09

Status: **VERIFY — owner local build and visual verification pending**.

React `main` commit `50135c4733e3633e1043fbdeb7249b7c385fb1f4` implements the owner's narrow navigation request:
- Rename legacy `Components` sidebar label to `App Administration` (provisional name chosen for its clear administrative purpose).
- Existing Recruitment entry in that lower section becomes a single non-expandable link to its existing `/recruitment` page; remove its duplicate Candidates child **from this sidebar section only**.
- Add equivalent non-expandable Transport and Housing links to `/transport` and `/housing`, backed by minimal authenticated workspace heading-only placeholders.
- Keep the Operations hierarchy (Recruitment/Candidates/Vacancies/Requests/Placements; Transport/Fleet/Drivers/Routes/Schedules; Housing/Properties/Residents/Occupancy/Maintenance) unchanged and above the administrative area.
- Leave other legacy Organization/Locations/System menu items and route definitions intact.
- No Off Canvas, permissions, backend, DB, schema or existing functional page modifications. Naming and new pages currently English-only; localization remains outstanding, not a completed 28-language claim.

Owner (React repo checkout):
```powershell
git pull
npm run verify
npm run dev
```
Check that App Administration entries do not expand, existing Recruitment administration page remains accessible, Transport and Housing show headings, Operations still expands and the duplicate Candidates sidebar item is gone. Do not mark DONE without owner acceptance.

## ADMIN-APP-PAGE-DESCRIPTIONS — 2026-10-09

Status: **VERIFY** (owner browser/local build pending). React `main` commit `aba18c08bfc3fd268eee1fb761aabe40cda9ea00`.

The three App Administration landing pages now show a module title and one short English description explicitly framing the page as an administration workspace. Recruitment uses its existing `/recruitment` page; Transport `/transport` and Housing `/housing` use the existing heading placeholder with an optional description prop. Operational pages, navigation labels, permissions, Off Canvas and backend are unchanged. The `App Administration` section label remains provisional, pending owner choice. New descriptions are English-only, and localization is pending; do not mark DONE.

Owner React checkout: `git pull`, `npm run verify`, `npm run dev`; inspect three admin pages, verify Operations headings and routes remain unchanged.

## APP-CONTROL-PANEL-LABEL — 2026-10-09

Status: **VERIFY**. Owner approved sidebar section name **App Control Panel**. React `main` commit `824e5d7c2b977678731f49874f4c8ba794b4d25c` changes only the heading string in `src/components/navbars/navbar-vertical/WorkspaceNavbarVertical.tsx` from `App Administration` to `App Control Panel`. Navigation entries, routes, pages, permissions, backend and other UI untouched. Owner: `git pull`, `npm run verify`, visual check. Label is currently English-only; localization remains pending.

## ADD-CANDIDATE-OFFCANVAS — 2026-10-09

Status: **VERIFY — local typecheck/build, visual and functional browser testing pending**.

React `main` commits `d0ecf55f66118e3fb176483e0b8bad940d8e66ac`, `5c7443aa8cea79577c7adc6df19b91ee3299b095`.

- Top navbar Add Candidate previously navigated to `/recruitment/candidates?add=1`, causing unwanted page change. It now opens shared Add Candidate panel without navigating.
- Candidates-page Add Candidate button opens exactly the same shared panel.
- Existing React Bootstrap (Phoenix theme) `Modal` changed to right-side `Offcanvas` (placement=end), 33vw width with small viewport minimum constrained to screen width. Close/disabled controls and original dev-only CV preview flow retained.
- Shared `AddCandidatePanelProvider` is mounted within `WorkspaceLayout`; source Candidates page no longer owns duplicate upload/modal state. Existing functional candidate list and routes left untouched.
- CV preview remains DEV-only, shows parsed identity/contact preview, does not persist candidate records. No Push/Create persistence action added. No backend, API, database schema, permissions or navigation section changes.
- Legacy `?add=1` query trigger removed from Candidates (not needed after shared panel); external bookmarks using it will no longer open the panel. Reconsider only if owner requests compatibility.
- No new translations necessary for the panel: existing `candidate*` keys reused.
- Source changes pushed without running the owner's local build, browser or tests. Never treat source-only review as DONE.

Owner update React checkout:
```powershell
git pull
npm run verify
npm run dev
```
Check header Add Candidate from Dashboard/Transport/Housing without page navigation; check same panel from Candidates page, ~33% width on desktop, backdrop/close behavior, CV PDF/DOCX preview, invalid file/error and loading-state close lock. Send verify output and visual observation to close block.

## DEV-CANDIDATE-SAVE-UI-1 — temporary local save action (2026-10-09)

Status: **VERIFY — owner React local build/browser test pending**.

Owner approved a temporary `Save Test Candidate` button rather than manual browser-console code. On React `main`:
- `src/api/resume-preview.ts` adds `saveTestCandidate(file)` calling the existing Express `POST /api/dev/candidates/intake` as PDF/DOCX raw body with encoded file name.
- `src/components/candidate/AddCandidatePanelProvider.tsx` adds a development-only button alongside existing Preview, blocks duplicate repeated clicks after successful save, displays returned candidateId, and resets save state upon changing/closing the file. Existing Preview stays non-persistent.
- `public/locales/*/auth.json` adds the two new visible strings for all 28 locales.
- No React route, Candidates table, Candidate Prototype, styling rules, backend API/schema or production UI changes.

IMPORTANT: This operation actually creates a candidate, contact, draft and link in local PostgreSQL. Use only consented/synthetic documents, do not repeat the same document until duplicates/reset are handled. Backend A2 is verified at test/build level (34 suites / 144 tests) but real DB insertion has not been manually checked. Frontend local verify and browser save are outstanding.

Owner: `git pull` in higa_systems_react, `npm run verify`, run dev React and Express, use Add Candidate → choose one test CV → Save Test Candidate, inspect candidateId and backend persistence. Keep VERIFY until owner reports local success. Only existing main, no branches or PR.

## DEV-CANDIDATE-PIPELINE-B — Recruitment table data (2026-10-09)

Status: **VERIFY — owner local React build and visual test pending**.

On React `main`, `src/pages/recruitment/Candidates.tsx` replaces static `candidateRows=[]` with dev-only `GET /api/dev/candidates` via the standard credentialed `apiRequest`. The current Phoenix table columns, filters, search, sorting, and pagination remain unchanged; country is intentionally empty (not guessed). Error shown via existing translation key. Refresh the page after saving a new Candidate to reload the list; live refresh and Candidate Prototype navigation are outside Block B.

Owner checks: `git pull`, `npm run verify`, restart React, open Operations → Recruitment → Candidates and verify the locally saved Endijs Runcis row. No React tests/build were executed by assistant.

## DEV-CANDIDATE-PIPELINE-C — Candidate detail click and real data (2026-10-09)

Status: **VERIFY — owner local checks/visual approval pending**.

On React existing `main` only:
- `src/pages/recruitment/Candidates.tsx`: Candidate name links to `/recruitment/candidates/:candidateId`;
- `src/Routes.tsx`: authenticated workspace route;
- `src/pages/recruitment/CandidateDetails.tsx`: working two-column dossier using real Candidate basic data, contacts and saved ResumeDraft fields (languages, summary, employment, education, skills, tools, certifications), fetched by authenticated `GET /api/dev/candidates/:id`.
The system Candidate Prototype route/file and its mock visual charts remain untouched. No ESCO normalization, editing, verification, permissions, data migration or changes to original CV handling. This view is development-only; no production Candidate read model exists.

Known constraints: other missing Candidate data stay absent; no portfolio/charts/verified facts are synthesized; the working detail page mirrors the Prototype's basic layout rather than embedding its static mock implementation. UI headings with no existing translation key in the shared namespace are temporary development labels, to be aligned in a later localized UI block.

Owner: `git pull`, `npm run verify`, restart React; click Endijs Runcis in Recruitment Candidates; inspect contact, 8 work entries and 22 skills against PostgreSQL source. Keep VERIFY until browser confirmation.

## End-of-day reconciliation — 2026-10-09

[Cross-project evidence and next steps](../../docs/operations/CANDIDATE_DEV_CHECKPOINT_2026_10_09.md). Owner reported React `npm run verify` green for Candidates list/detail and supplied screenshots showing loaded Candidate records and live two-column dossier pages based on saved CVs. Earlier *owner visual verification pending* statements in the Candidate-only A/B/C entries are superseded by this browser evidence; the development listing and detail work, while broader React regression and production acceptance remain open. Observed defects: 429 rate limit after five CV requests per 15min (backend), stale green Preview banner alongside red error (React); raw `candidateDevSaved` locale key observed earlier (recheck if it persists). Fixes proposed but **not implemented**. Design and prototype unchanged by this documentation checkpoint.

## DEV-CV-HARDENING-1 — stale Preview success correction (2026-10-10)

Status: **VERIFY — owner local/browser check pending**. React `main` commit `16cf8ffc61b243417a1754252e5fc0b7b05d1e91` clears a previously successful Preview before starting `Save Test Candidate`, so any following API failure cannot leave the old green preview next to the new red error. No style, route, locale, Candidate detail, API contract or production change. Owner must `git pull`, `npm run verify`, restart React, test successful Preview then failed Save (or an API error); verify previous green preview is gone. No agent-run local verification.

## CANDIDATE-ESCO-RESEARCH-1 — three visual blocks (2026-10-10)

Status **VERIFY — owner local/browser checks pending**. React main CandidateDetails displays existing AI Skills/Tools unchanged, then independent ESCO Occupations and ESCO Skills blocks sourced from dev-only Candidate detail `esco` response. Phoenix badges temporarily distinguish Occupation, Direct Skill, and Occupation-Related ESSENTIAL/OPTIONAL; URI opens official concept, tooltips show raw source title/relation. Empty state says no exact proposals. No frontend benchmark, CV comparison scoring, verification/intake, persistence, CSS framework change or migration. Owner to `git pull`, `npm run verify`, reload and inspect several candidate pages. If results are sparse, treat as evidence of strict exact-label search, not evidence that candidate lacks skills.

## CANDIDATE-ESCO-TIMELINE-1 — per-work-period prototype (2026-10-10)

Status **VERIFY — owner's npm run verify/browser check pending**. Candidate Details now renders each `workHistory` item as its own responsive row: left original title/employer/dates/description; right this row's proposed ESCO occupations, direct skills and a collapsed Related Skills details block. CV-wide AI Skills/Tools displayed separately below, along with collapsed ESCO unassigned direct matches; Education remains; Certifications removed only from this experimental view, not deleted or migrated. Contacts/languages remain existing side panel (fixed-panel UX deferred). No ESCO or final Candidate UI architecture changes; colors temporary; no assertions of verified competency. Check mobile/desktop layout and all saved CVs locally.
