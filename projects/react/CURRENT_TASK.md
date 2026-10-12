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


## 2026-10-10 CV intake and AI–ESCO research

CV intake UX experimental change (2026-10-10): DEV Add Candidate now selects one PDF/DOCX then one Save Test Candidate action; Preview button and intermediate success panel removed. Offcanvas closes on successful existing synchronous POST; errors remain visible, in-flight request still blocks closing. This is NOT a background queue: users cannot immediately continue during parse yet. Candidate sidebar displays `AI cost: Not recorded`, because historical per-candidate usage/billing is not persisted. Owner verify pending.


## AI ↔ ESCO Research V1 (2026-10-10)

2026-10-10 AI↔ESCO explicit research — STATUS VERIFY (owner local tests/browser pending). Candidate Details has Run AI ↔ ESCO research button, explicit POST; while running shows loading and maintains existing lexical result; on success displays per-work-period AI-selected ESCO Occupations and AI Proposed Skills, with full Related graph skills still collapsed. Response is ephemeral client state, lost on reload. Errors shown without erasing prior results. Old Candidate GET remains no-cost lexical preview. AI cost line unchanged ('Not recorded'). No upload workflow changes in this block. Run npm run verify, test actual saved CV.


## ESCO concept tooltip and cost research

2026-10-10 ESCO Popovers + iCosts checkpoint — **VERIFY, owner npm run verify/browser pending**. Experimental Candidate Details now uses shared `EscoConceptBadge` with React-Bootstrap Popover on hover/focus for all ESCO occupation/skill/related/unassigned badges. Fetches official description lazily through new read-only `GET /api/esco/concepts/:id/description` using active dataset's EN/NL DESCRIPTION/DEFINITION/SCOPE_NOTE, cached per concept in React. Displays clear fallback if description absent; never uses whole CV text as description. Existing graph and migrations unchanged; added endpoint route test. Cost: no feature change; current historic Candidate says `Not recorded`; accurate per-candidate iCosts require collecting operation model + actual input/output tokens, pricing tariff/version and operation ID at execution; Billing can reconcile totals, cannot reliably allocate historical charges to candidate. No API/admin key requested. Visual and real description content needs owner check.


## Occupation-first ESCO research UI checkpoint — 2026-10-10

Status **VERIFY — owner npm run verify/browser pending**. Removed unreachable `Open ESCO concept` link from read-only badge popover; official ESCO description remains on hover/focus. Removed expanded Related ESCO Skills details blocks from experimental Candidate Details UI only. Related graph remains in backend `candidate.esco.experiences[].relatedSkills` and unchanged ESCO tables for future graph-distance/similarity research; did not delete data or change ESCO architecture. Main view emphasizes work-period occupation proposals and relevant AI skill proposals. **Do not yet invent match percentages, mark AI proposals Confirmed, or claim deterministic repeatability**. Next research candidate occupation recall and rerun variation: evaluate leaf occupation concepts vs ISCO 10 major groups (taxonomy, not 10 ESCO professions), generate controlled candidate set, AI selection evidence and hierarchy distance, then propose a scoring contract before implementation. No semantic-matching backend changes in this checkpoint.


## Occupation Discovery instruction experiment and diagnostics — 2026-10-10

Status **VERIFY — owner local npm run verify and live browser run pending**. User showed CV: Warehouse Employee → Warehouse worker, Electromontage → Electrician; Manager-consultant with retail sales/customer advice/payment tasks → no occupation. Changed only OpenAI discovery prompt to prioritize documented duties, realistic Preferred/Alternative occupation search phrases and avoid unsupported managerial seniority; no canonical ESCO mutations. Experimental read-only AI response adds per-work-history `diagnostics` with bounded occupationQueries, retrievedOccupations (real concept ID, label, matchedTerm) and selectedOccupationIds; React CandidateDetails displays them only inside collapsed Research Diagnostics for that same period. Added regression assertions for fixture timeline isolation. Nothing is persisted; results remain ephemeral and paid calls remain explicit. This is observability, NOT guaranteed improvement; owner to pull BOTH Express and React main, run both `npm run verify`, restart services and test one CV. Do not infer exact reason for a failed match without diagnostics.


## Esco Intelligence Bridge V1 — 2026-10-10

**VERIFY; owner local checks pending.** Added `src/esco/bridge/work-context.ts` as internal typed, read-only intermediary between Uno AI and existing ESCO graph. Existing `candidate.esco.discover@1` now emits **structured interpretation of one work period** (`coreActivities`, `responsibilities`, `equipment`, `workingConditions`, `evidenceGaps`) together with bounded occupation/skill ESCO lookup phrases, avoiding another model call. Work context is stored **only in ephemeral research API response** (`candidate.esco.experiences[].interpretation`) and displayed in collapsed `Uno Interpretation` for each Work Experience separately, alongside existing Research Diagnostics/ESCO suggestions. No DB/schema/migrations, ESCO alterations, auto confirmations, persistent AI decisions, billing changes or changes to candidate upload. Updated test mocks for typed discovery output and assertion on context; all test/build results **unverified pending owner `git pull` and `npm run verify` in BOTH Express and React**. Isolation still limits processing to first 8 work periods as previously. This is an observability/interpretation foundation, not proof of improved matching or semantic search. Compare interpretation, retrieved candidates and final ESCO selections on same CV to isolate failure source.


## ESCO Bridge V1 — Evidence Mapping checkpoint (2026-10-10)

**VERIFY, owner local checks pending.** Created `src/esco/bridge/evidence-mapping.ts`: deterministic conservative mapping of already extracted `ResumeDraftV1.skills` and `.tools` to one `workHistory[sourceIndex]` ONLY if a normalized whole-word/whole-phrase literal occurs in that job's title/description. No inference from other periods, no fuzzy semantic claims, no new AI calls, and no confirmations. Attached `experience.evidence` to temporary AI ESCO response, added collapsed `Uno Evidence Mapping` to CandidateDetails. Up to two source-linked skills feed existing bounded SKILL label retrieval alongside max-six AI skill queries; CV-wide tools are displayed as evidence but not automatically treated as ESCO skills. New tests check ETL/Docker vs Forklift period isolation and non-match of SQL inside NoSQL. No migrations, ESCO graph/source changes, persisted candidate skill decisions, or iCosts changes. Limitations: paraphrases and abbreviations without literal matching are deliberately not assigned; evidence strings currently reflect matched CV-wide terms, not character offsets. Owner should pull BOTH Express and React main, run `npm run verify` for each, and browser-check per-period Uno Evidence Mapping and skill proposals. Do not claim tests passed until owner confirms.


## Candidate ESCO lexical evidence diagnostic display (2026-10-10)

**VERIFY — owner local/browser checks pending.** Approved limited display-only scope on existing React `main`. In `src/pages/recruitment/CandidateDetails.tsx`, existing expandable `Research Diagnostics` now displays optional backend `diagnostics.evidenceDiagnostics`: official description availability, textual shared words, context words absent from the ESCO description, and the diagnostic assessment. Uno-selected candidate marked in the same list; Express now guarantees selected occupation has a diagnostic even when ranked beyond the initial 12. UI explicitly says lexical overlap is not professional proof and absent words are not contradictions. Existing export/copy of visible page text can capture the expanded diagnostic. No separate report endpoint, no new button, changes to selection, persistence, design system or Candidate architecture. React commit `2448034c`; Express commit `ae20ffdf`. Owner needs `git pull` and `npm run verify` in both repos, rerun dev research and expand `Research Diagnostics`; confirm selected concept evidence appears. Agent did not execute local checks. Status remains VERIFY.


## ESCO OCCUPATION MATCH SCORES UI — 2026-10-10

**VERIFY — committed to existing React main; owner npm run verify and browser check pending.** Experimental `src/pages/recruitment/CandidateDetails.tsx` now reads optional `matchScore` from Express AI occupation proposals and displays a 0–100% match value and compact progress line per period/occupation. Removed obsolete visible Uno Interpretation, Evidence Mapping and Research Diagnostics sections from the Candidate Details experimental view; there is no deletion of ESCO graph, persisted Candidate/Resume data or shared APIs. When AI results are present, the Skills section explicitly says occupation confirmation is pending and hides old lexical direct skills for that AI view. Initial read-only lexical preview remains separate and unchanged upstream; it has no match scores. Experimental classify button continues calling existing authenticated dev-only endpoint and old response contract remains compatible via optional score. No real confirmation workflow has been added.

**Owner verification required:** React main `npm run verify`, browser inspect saved candidate with two work periods, classify and verify percent/descending order and absence of diagnostics, no invented skills, responsive layout and empty proposal cases. Backend companion block is in Express CURRENT_TASK. No agent local build/browser pass claimed; status VERIFY.

## VERIFY — Candidate Details AI↔ESCO diagnostic display (2026-10-11)
Owner approved connecting supplemental AI Interpreter and HIGA Knowledge diagnostic blocks to the EXISTING per-period research UI; no visual restyling, queue, migrations or new API. React `src/pages/recruitment/CandidateDetails.tsx` on `main` commit `65fa6fd`: after the existing manual `Run AI ↔ ESCO research` action, each work period shows AI-selected occupation count and IDs, source title/description availability, lexical occupation count, and explains that the AI API returns no narrative reasoning. Adds an honest HIGA Knowledge no-mapping notice (no fabricated data, no HIGA API call). Fixes presentation regression: existing source-linked lexical ESCO direct skills stay visible when AI occupation research is displayed; these are NOT confirmed skills. No backend/provider change; no automatic AI call/queue; no tests or owner visual verification performed here. Owner next: `git pull`, `npm run verify`, open Candidate Details, click research, inspect occupations/diagnostics/source skills and report result. Status VERIFY.

## VERIFY — Candidate AI ESCO Related Skills technical view (2026-10-11)
React main `src/pages/recruitment/CandidateDetails.tsx` commit `2525448`: adds `ESCO Related Skills · occupation-based proposals` block beside existing work period ESCO Occupations, direct source-linked lexical skills, AI Interpreter and HIGA Knowledge boundaries. Related skills from manually executed AI research display separately as collapsible `ESSENTIAL` and `OPTIONAL` groups with ESCO concept badges; clear caveat they are official occupation associations, NOT candidate proven competencies. No aesthetic redesign, no automatic parsing or queue, no new calls except existing research action. Status VERIFY: owner `git pull`, `npm run verify`, then click existing Run AI ↔ ESCO research and inspect e.g. corporate trainer related skills. No local build/browser verification asserted. 

## VERIFY — Show read-only independent SQL ESCO skill candidates (2026-10-11)
React main `src/pages/recruitment/CandidateDetails.tsx` commit `bea5d79` displays the count and expandable official ESCO ID badges from `diagnostics.skillDescriptionCandidates` under existing per-work-period AI Interpreter diagnostics after manual `Run AI ↔ ESCO research`. Explicitly unverified SQL DISCOVERY candidates, not personal skills, not selected AI matches. No CSS redesign, queue, persistence, permissions or new API route. Owner `git pull`, `npm run verify`, restart backend and inspect existing CV result; pending latency and relevance measurements. Status VERIFY.

## VERIFY — ESCO semantic skill research display (2026-10-11)
React main commit `9bec4fe`: CandidateDetails AI research section now distinguishes ESCO Essential, Optional and Independent ESCO Skills selected by backend semantic validation. Independent suggestions are visible even where there are zero proposed ESCO occupations. Renamed display to semantic proposals and explicitly says NOT confirmed candidate competencies. Existing SQL candidates diagnostics retained as unverified discovery. No new pages, CSS migration or HIGA Knowledge persistence. Run `git pull`, `npm run verify`, inspect CandidateDetails manually with existing CV. Status VERIFY pending owner results.

## VERIFY — Retire ESCO UI from Candidate Details (2026-10-11)
Owner explicitly stopped ESCO Matcher experiments due to repeated irrelevant skill proposals and instructed Candidate flow to rely solely on AI Interpreter. React main commit `f8735dc`: removed CandidateDetails ESCO badge, action button, classification API call, occupations, skill proposals, and SQL diagnostics; shows work history and CV-wide AI Skills/Tools already persisted in ResumeDraftV1. No invented per-period skill attribution, taxonomy/graph writes, translations or descriptions yet. AI Skills remain CV-wide. Does not remove shared ESCO source modules or database. Owner must run `npm run verify` and inspect existing candidate. Status VERIFY.


## ARCHITECTURE RECONCILIATION — Candidate Details / Interpreter-first (2026-10-11)

Status: **DISCUSSION / DESIGN** for next functionality; existing UI removal **VERIFY**. React `f8735dc` removes active CandidateDetails ESCO research button, classification calls, ESCO badges and SQL diagnostics, leaving work history and persisted CV-wide AI Skills / AI Tools. Do not reinstate the old UI or depict CV-wide skills as per-job verified facts. Express `799d239` returns stored Candidate draft without synchronous ESCO lookup; legacy experimental endpoint remains outside active UI. Owner local `npm run verify` and browser regression confirmation pending.

Next proposed UX is evidence-oriented per-work-period Activity / Skill review, then explicit HIGA Knowledge mapping/proposal statuses. No new frontend API, UI design, catalog records, migration or autoapproval implemented by this documentation checkpoint. Preserve original CV source, corrections and distinct candidate vs global Knowledge approval. Candidate/vacancy shared concept identity is a future contract, not a current React feature.


## ESCO DECOMMISSION — front-end cleanup, VERIFY (2026-10-11)

Main `8befbcb` deletes unused `src/components/candidate/EscoConceptBadge.tsx` (legacy `/api/esco/concepts/:id/description` UI); main `851eb52` removes leftover ESCO copy from CandidateDetails. `npm run verify` on owner's local React not yet confirmed after these commits. Additional content audit pending. Backend migration is a separate destructive step and must not be deployed without restorable backup.


## FINAL ESCO RESIDUAL AUDIT — 2026-10-11

GitHub recursive `main` trees scanned for 13 HIGA repositories (excluding `phoenix-react-reference`). No filenames with ESCO remain in active Express/React/Native or legacy HIGA repos. Deep per-file content scan of 115 Express text files (code, tests, config, SQL) found **zero ESCO tokens**. Deep per-file content scan of 153 React text files found **remaining ESCO references in 28 `public/locales/*/auth.json` translation files**, concentrated in `candidateResumeDescription`, `candidateResumeScopeDescription`, `candidateOccupationMasterDescription` (Portuguese substring false positives must not count as ESCO tokens). No actual active React TS/TSX ESCO mentions found in this scan. **React is NOT text-clean yet.** `higabase_plans` still has 7 ESCO-named docs, `higabase_states` 1 `pages/esco-ai/README.md`, and standalone `higa-esco-extractor` remains its own repo. React verification since badge removal not supplied. This entry reports findings only; translations and documentation not silently deleted. Follow-up: replace obsolete localized ESCO copy with accurate HIGA-independent wording in all locales, test React; reconcile/remove obsolete active ESCO docs while retaining decision/audit history; clarify whether to archive/delete standalone extractor repository.


## ESCO residual removal — locales CLOSED / React VERIFY (2026-10-11)

All 28 `public/locales/{lang}/auth.json` files were updated on React `main`: stale ESCO copy in candidate CV and occupation master descriptions replaced with HIGA-native wording, restored in each native language. All 28 JSON payloads were parsed and scanned for whole-word `ESCO` and each returned **zero**. `EscoConceptBadge.tsx` was previously deleted. Local React `npm run verify` is still **pending owner**; no green build claimed. The standalone `higa-esco-extractor` is a separate repo and remains untouched; removing its repository is a separate account-level destructive operation.


## Candidate Details — work history readability (2026-10-11)

Owner demonstrated the dev candidate dossier from uploaded CV. Issue 1: `src/pages/recruitment/CandidateDetails.tsx` rendered each `job.description` as one unbroken paragraph, making multiple CV responsibilities unreadable. React `main` commit `4d3ceae` renders existing work-history description as a visually separated bulleted list, splitting at newline or sentence boundary; source text remains unchanged. This is **presentation-only** and does NOT turn sentences into canonical HIGA responsibilities, nor assign experience-scoped skills or update parser/DB. Issue 2: the CV parser's earlier-employment entries (PTC CEO and Elkor supervisor without dates/detail) and education extraction completeness require separate investigation of source CV ambiguity versus parsing loss. **VERIFY**: owner `git pull && npm run verify`, inspect Candidate Details page; no green verification supplied yet. Future HIGA concept badge architecture stays DESIGN DRAFT.


## Phoenix Candidate Prototype / Resume mock UI (2026-10-11)

Owner requested a practical visual mock-data implementation at `/system/candidate-prototype/resume`, adhering strictly to existing Phoenix design standards rather than introducing a separate design system. React `main`: `src/pages/system/CandidateDossierPrototype.tsx` updated (`60875b1`, `61537dc`, `90ecbed`) to replace the old split-column occupation placeholder with a compact **Profile Overview** section and **time-bounded Work Experience accordion**. Donut uses the existing ECharts/Phoenix CSS token-based chart (`src/components/charts/CandidateOccupationProfileChart.tsx`, commit `f395360`), now accepts mock category counts and fires chart-slice selection. Overview exposes Responsibilities, Skills, Knowledge, Tools & Technologies, Qualifications, and selected sample tags. Expanded job row shows timeframe/company/title, concise bullet description (left ~5/12), and per-job Phoenix Nav pills (right ~7/12): Responsibilities, Skills, Knowledge, Tools. Collapsed PTC and Elkor entries preserve unknown dates (do not fabricate timelines). Sidebar keeps existing prototype menu/profile and adds Languages/Education sections. **All display data are mock**; category counts illustrative, not generated or sourced from real candidate DB. No Prisma/API/AI/parser integration, screening or matching work. Existing route/layout and Phoenix/react-bootstrap widgets retained. **Verification pending owner**: `git pull && npm run verify` and inspect `/system/candidate-prototype/resume`. If compile/type errors appear, fix immediately with accompanying MD update. Separate candidate details page unchanged.


## Phoenix Profile Overview refinement — mock only (2026-10-11)

Owner reviewed screenshots of `/system/candidate-prototype/resume` and requested top overview refinement only. React `main` commits `c0883f2` and `fc6fcd7`: split `Profile Overview` title into its own Phoenix `border-bottom border-translucent` header; replaced grey button-like category rows with flat border-separated clickable legend rows, muted adjoining-hue five-segment palette shared exactly with ECharts, color stripe and Feather icon for each; added **All** as sixth selection (not sixth graph segment) showing all category mock tags, with per-group categories still available through click on donut slices or rows. Chart/legend/tag panel alignment changed to a stable `5/12` + `7/12`, with nested `6/12` category/tag columns and responsive stacked layout. Mock category counts unchanged; do not claim real candidate data. No backend/schema/AI changes. **VERIFY** owner to run `git pull && npm run verify` and inspect light/dark/browser layout; no local verification confirmed yet. Exact color values are local chart palette for mock prototype, remaining components and spacing use established Phoenix tokens/Bootstrap conventions.


## Profile Overview compact/rainbow + stock-style selector (2026-10-11)

Owner requested compact header, smaller graph, more space for badge content, no duplicate `All knowledge groups` heading or helper caption, visually blended rainbow-like donut, and a stock/Phoenix company-list-like selector. React `main` changes: `CandidateOccupationProfileChart.tsx` commit `d5ee088` adds subtle ECharts segment gradients (still mock data, still ECharts); `CandidateDossierPrototype.tsx` commits `a93a9f5` and `9876317` shrink header to `py-2`, chart to 192 px and narrow `xl=3` chart column, increase tags area, remove redundant labels, and replace list of thin underline buttons with a compact stock-style selection list (30px tinted icon tile, bold label, count at right, Phoenix `bg-primary-subtle` active state). Preserve existing Phoenix/Bootstrap classes and Feather icons. Mock-data interactions only. **VERIFY**: owner should `git pull && npm run verify` and review the layout. No local build confirmation and no browser access to local URL from GitHub context.


## Candidate Profile Overview order / donut thickness — owner screenshot refinement (2026-10-11)

Owner specified visually: category selector must come **FIRST (left)**, then a centered donut, with badges/content to the right; remove dividing line between selector and donut, keep current overall column proportions, double donut **ring thickness inward** (not outer diameter), tighten excessive header vertical padding. React `main`: `src/pages/system/CandidateDossierPrototype.tsx` commits `9369785` and `2f8fb22`: selector/chart/tags column order `xl=3/3/6`, no boundary between selector and graph, chart centered using Phoenix flex utilities and 216px chart container, reduced header from `py-2` to `py-1`. `src/components/charts/CandidateOccupationProfileChart.tsx` commit `80ada3b`: ring radius inner 62% → 36% while outer remains 88% (~double thickness). Mock data and work-experience layout unchanged. **VERIFY pending:** owner `git pull`, `npm run verify`, inspect header spacing on local browser; source was updated but no local build or screenshot confirmation available.


## Phoenix stock/event reference correction (2026-10-11)

Owner supplied Phoenix original reference URLs `https://phoenix-react.prium.me/dashboard/stock` and `https://phoenix-react.prium.me/apps/stock/stock-details` plus a Phoenix event-style card screenshot, requesting consistent source-template styling for both selector and Work Experience. React `main` commit `c6057c4` updates only mock Candidate Prototype: category selector uses bordered rounded Phoenix-like card rows with selected primary border and subtle background; Work Experience header uses a compact date/timeframe tile on the left, clear role/company details centrally, and a Phoenix secondary expand/collapse control on the right; expanded description and knowledge tabs preserved. No change to mock values, graph, API, parser, schema or DB. Live reference pages did not resolve through current web fetch, so code implements the user's screenshots and existing Phoenix Bootstrap class idioms rather than claiming exact upstream source copy. **VERIFY pending**: owner to `git pull && npm run verify` and inspect local prototype; no local execution reported.


## Root-cause fix: Phoenix global `section` padding (2026-10-11)

Owner DevTools screenshot established `theme.css` global media query at >=992px: `section { padding-top:3.5rem; padding-bottom:3.5rem; }` (also base `section` padding 2.5rem). Previous repeated local `py-1` changes could not reliably remove root padding because global theme `section` specificity/cascade applies to literal semantic tag. React `main` commit `f52809f` replaces **only CandidateOccupationProfile's outer `<section className="border-bottom">` with `<div className="border-bottom">`**, removing unwanted page-level ~112px extra vertical padding without changing Phoenix global CSS, chart, rows, work experience, backend or mock data. **VERIFY pending:** owner `git pull`, `npm run verify`, inspect header spacing and computed style; no successful local build claimed.


## Phoenix original event-card interaction correction (2026-10-11)

Owner rejected bespoke experience layout/button, requesting fidelity to original Phoenix Stock Details / Upcoming Events card pattern. Upstream React live demo URLs could not be fetched directly (404 in accessible web), but public Phoenix HTML documentation displays the event card content structure; owner screenshot is the visual reference. React `main` commit `6dc092c`: replaced separately clickable `Details/Less` CTA with **one entire full-width semantic button for the event-style work experience header** (aria-expanded; no nested buttons). Left date tile explicitly shows `From` / `Till` as two rows for known `Feb 2021 – Nov 2025`, unavailable dates are honestly stated rather than fabricated; title/company use existing Phoenix `text-body-emphasis` / `text-body-secondary` and `fs-9`; right edge only has chevron. Existing expanded body and mock data preserved. No custom CSS or Phoenix theme overrides; no production API/DB modifications. **VERIFY** owner must `git pull && npm run verify` and check actual browser layout. Do not claim exact copied Phoenix React stock component because upstream source wasn't accessible.


## Exact stock Phoenix EventItem inserted, no custom experience component (2026-10-11)

Owner explicitly ordered inserting an original template component unchanged first, then directing which candidate values go to which fields. React `main` commit `dba041c` replaces the custom `CandidateWorkExperience` accordion mock UI inside `src/pages/system/CandidateDossierPrototype.tsx` with direct rendering of existing `src/components/list-items/EventItem.tsx` using unchanged original `src/data/eventsData.ts` `events` samples. **Original EventItem source, styles and dataset are untouched**. This is an authentic existing Phoenix event-list item from this project's template, not an asserted match to the user screenshot of the Stock Details event card; exact Stock Details original TSX has not been identified in this repository. The current demo intentionally displays stock Phoenix event titles, not mapped candidate employment fields, pending the owner's mapping instructions. Profile overview and sidebar unchanged. No backend/API/DB modifications. **VERIFY pending**: owner `git pull && npm run verify`, view local `/system/candidate-prototype/resume` and identify desired field positions. If the owner means a different Phoenix component from the screenshot, request its actual source file rather than recreating it.


## CANDIDATE-WORK-EXPERIENCE-PHOENIX — 2026-10-11

Status: **VERIFY — owner local build and visual check pending**.

Owner-requested narrow prototype UI change on React `main`: `src/pages/system/CandidateDossierPrototype.tsx` replaces the three demo `EventItem` records in Work Experience with one card using the original Phoenix `UpcomingEventCard` JSX/classes and Stock Details demo content. Reference: `phoenix-react-reference/src/components/cards/UpcomingEventCard.tsx` and `src/data/stock/stockDetails.ts`.

- Keeps existing Work Experience heading and Candidate Resume route; no backend, storage, API, navigation or CSS changes.
- Deliberately **does not** include Phoenix `EventOffcanvas`, the title click handler or any Canvas/offcanvas overlay, per owner request.
- The three original event/film demo records are no longer rendered in Work Experience.
- Phoenix reference avatar binaries were not available through the GitHub connector; three initial-based placeholders are used inside the Phoenix avatar-group area. This is **not pixel-identical** to the original reference and remains open for owner acceptance/replacement by original image assets. No additional CSS was created.
- This is stock demo data, **not** real candidate job experience. Interested/Going retain the Phoenix visual buttons without backend actions.
- Do not mark DONE without owner `npm run verify` and browser visual approval.

React commits: `8211c1fa363511bd3d991f49b5801bd8e9885c2d`, `fd58042a1d5d811586bf0d2a7862e3109b7cab03`.


## CANDIDATE-PROFILE-CRM-DONUT — 2026-10-11

Status: **VERIFY — owner local build and visual inspection pending**.

Owner requested the Phoenix CRM Dashboard donut chart shown at `/dashboard/crm` in the existing Candidate Resume profile overview. Inspected canonical reference `phoenix-react-reference/src/components/charts/e-charts/ContactsBySourceChart.tsx` and `src/components/modules/crm/ContactsBySource.tsx`.

React `main` commits `d89bb406538f0854f8faca9faaf46182f2fd9825` and `2a37fc792942c06b931a53b57f3e4781c2593435`:
- Switched `CandidateOccupationProfileChart` to Phoenix CRM pie radius 55%-90%, 3px body-background separators, theme palette and dark/light treatment; removed custom gradient sectors and custom fixed hex palette.
- Used Phoenix `100px` central `bg-primary-subtle` circle with total in `text-primary-dark`, and 245px chart height.
- Preserved candidate-specific 5-category values, existing category buttons, chart click-to-filter behavior and the rest of Resume; chart total is derived from candidate mock concepts, **not** copied Phoenix demo value 560.
- No new CSS, dependencies, backend/API, schema, or other routes. Existing chart uses ECharts CanvasRenderer already present; it is **not** React Canvas UI.
- The category-icon background uses a CSS `color-mix()` tint of the matching theme color; this is a local style expression rather than an imported standalone Phoenix legend component. Full rendered parity is not yet verified.

Owner: run `npm run verify` on updated React main; visually check light/dark, five sectors, center circle and clicking chart sectors/category buttons. Mark DONE only after owner confirms.


### CANDIDATE-WORK-EXPERIENCE-PHOENIX — build regression fix (2026-10-11)
Owner ran `npm run verify` and reported TypeScript TS2307 for `dayjs` and `dayjs/plugin/customParseFormat` in `CandidateDossierPrototype.tsx`. Root cause: original Phoenix date parsing import was copied without an installed `dayjs` dependency. React main commit `adefa150413043c9cdd522aa5f2c521c99991395` removes the undeclared imports/plugin and renders the existing static demo date directly (24 / Mar, 2024). No dependencies, CSS, backend, or routes changed. **VERIFY** pending owner's renewed `npm run verify`; no green build has been claimed.


## CANDIDATE-PROTOTYPE-V2-PHOENIX-SOCIAL — 2026-10-12

Status: **VERIFY** — user local build, asset loading, and browser parity pending.

Owner-approved new, separate prototype surface, built block-by-block. React `main` adds `/system/candidate-prototype-v2` with `WorkspaceLayout` and `AuthGate`. The existing `/system/candidate-prototype/resume` was not changed.

Left column: `Col xl={3}` (25% at xl+) contains the Phoenix Social Settings `SocialProfileCard` mock card composition from `phoenix-react-reference/src/components/cards/SocialProfileCard.tsx`: original Ansolo Lazinatov, u/hansolo, 1297 Followers and 3971 Following, cover and avatar. Right `Col xl={9}` (75%) is intentionally empty. Smaller breakpoints use Phoenix/Bootstrap 12-column stacking. No new CSS, backend, API, schema, invented UI copy, or candidate data.

Image references are the original Phoenix demo's public static artwork URLs (`generic/59.png` and `team/20.webp`) rather than copied binaries; browser loading/cover availability must be checked by owner. The card is a static read-only mock, **not a full copy of working CoverUpload/AvatarUpload controls**; upload interactions are not authorized by this block. The layout/classes follow SocialProfileCard; confirm visual fit at the narrower 25% column.

React commits: `7c4d917710e99c52d309935fbfc4c03571ca2853`, `763cdccc0ee99c5b87b6dee0285efc9824519f5a`. Owner: `git pull --ff-only origin main`, `npm run verify`, open `/system/candidate-prototype-v2`. Do not mark DONE until verified.


## CANDIDATE-PROTOTYPE-V2-STOCK-NAV — 2026-10-12

Status: **VERIFY** (owner local `npm run verify` and browser inspection pending).

Owner-requested Phoenix Stock Dashboard sidebar-card navigation adapted as a persistent left-column navigation below existing Social Profile card. Reference: `phoenix-react-reference/src/components/modules/stock/TopStocks.tsx` with `Nav.Link as={Card}` + `company-card`, and `src/components/cards/TopStocksSidebarCard.tsx`. React main commits `6f3cc455007a5ba4c1638cc6a26338ed5c7acba3`, `51a404a8c530499e9bf4724b01bed1a8f85508ce`.

- Exactly two sidebar cards: Dashboard and Resume, icon + text; no stock prices/numbers. Styling uses existing Phoenix stock card classes and react-bootstrap primitives.
- Layout stays left 25% / right 75% at XL+. The left profile and navigation remain mounted in parent layout, while the right Outlet renders distinct React Router child pages rather than React Tab panes.
- Routes: `/system/candidate-prototype-v2/dashboard` (empty), `/system/candidate-prototype-v2/resume` (empty); index redirects to dashboard. This is **client-side route navigation** with a browser URL change and route outlet, not a full page network reload.
- Existing candidate-prototype (v1) pages and API unchanged. No CSS, backend, storage or candidate data.
- Visual matching at 25% width, active state and browser route navigation not yet verified by owner. Do not mark DONE before acceptance.


## CANDIDATE-PROTOTYPE-V2-NAV-COMPARISON — 2026-10-12

Status: **VERIFY** — owner local build and visual review pending.

Owner requested *both* compact Phoenix sidebar candidates displayed for comparison under Social Profile, **without active navigation links**. On React main commit `da37671593e0b0314e2fcee6549c77e2ffc7b16e`, `src/pages/system/CandidatePrototypeV2.tsx` replaces the oversized Stock cards with two independent, inert displays: (1) Phoenix `vertical-tab` (as used by existing `CandidateDossierPrototype`) and (2) Phoenix `navbar-nav` style. Both show Dashboard and Resume, with the first item styled active for visual comparison. No NavLink, href, onClick, state, or navigation action is attached to these displays. Existing nested Dashboard/Resume routes remain defined, but the comparison controls do not open them. Right-hand `Outlet`, left profile card, and 25/75 grid remain unchanged. No CSS, libraries or backend changes.

Verification pending: owner `git pull`, `npm run verify`, `npm run dev`, compare both visual variants. Do not mark DONE before owner review.


## CANDIDATE-PROTOTYPE-V2-SUBTLE-NAV — 2026-10-12

Status: **VERIFY** — owner local build and browser check pending.

Owner approved replacing the two inert prototype navigation comparisons with a single compact Phoenix Subtle Buttons variant, with working links. React main commit `90e590790a313b86a9a4f5b2100f8394c3e304b8` updates only `src/pages/system/CandidatePrototypeV2.tsx`:
- Two `NavLink` controls styled with existing Phoenix `btn btn-sm` plus active `btn-subtle-primary` / inactive `btn-subtle-secondary`, with Feather icons.
- Dashboard navigates to `/system/candidate-prototype-v2/dashboard`; Resume to `/system/candidate-prototype-v2/resume` through existing nested routes and `Outlet`.
- Removes both no-link visual comparison sections. Persistent left profile card, 25/75 layout and intentionally empty right-hand child pages are preserved.
- No new CSS/dependencies, backend, candidate model, or other page changes.

Owner verification: `git pull`, `npm run verify`, `npm run dev`; inspect size/appearance, active state, URLs and persistent left panel. Do not mark DONE without owner verification.
