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
