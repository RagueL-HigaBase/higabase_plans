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
