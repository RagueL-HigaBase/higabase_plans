# Workspace Recovery — Diverged Local and Main History

Status: **VERIFY — owner local visual check pending**
Date: 2026-10-08
Scope: frontend restoration only, no backend domain schema, no permissions changes.

## Root cause

The owner previously worked on commit `35456e7`, which contained UI changes not present in `main` at the time the owner was instructed to switch to `main`. Moving to main consequently hid the Candidate Dossier prototype, email verification/recovery screens, the masked PIN presentation and a more detailed Organization/Locations navigation. These features were not deleted by the most recent CV preview commits; they were absent from the earlier main history.

This recovery reads historical source **without creating or checking out any branches**. All changes are committed directly to `main`.

## Restored UI and wiring

1. PIN masking: restore the original four-digit Phoenix layout, tooltip and accessible digit labels. Use `type="password"` for reliable masking across browsers (not CSS-only WebkitTextSecurity).
2. Email verification: restore the existing backend contract in frontend API, explicit `/verify-email` page, resend flow, `verificationEmailRecoveryLink` in Sign In, and sign-up-to-verification routing. Restore CompleteProfileForm and handle `profile-required` before company setup (backend already requires this).
3. Candidate Dossier Prototype: restore the original `src/pages/system/CandidateDossierPrototype.tsx`, original `CandidateOccupationProfileChart.tsx`, its ECharts dependencies and matching lockfile. Route `/system/candidate-prototype` with dashboard/resume children and SYSTEM navigation item. This remains static prototype/test content, not canonical Candidate storage.
4. Organization navigation: restore **Organization** (Dashboard, Departments, Team) and separate **Locations** (Main Location → Overview, Departments, Team). Keep the current global Dashboard and Recruitment menu/routes. Restore the Organization-level placeholder pages and Location Departments route. Do not delete the existing Location shell, its stock tabs or additional connections/invoices/documents/settings routes.
5. i18n: merge missing prior PIN, verification, Candidate Prototype and Location Department labels into all 28 existing locale files. Preserve current translations and new Candidate list/CV preview keys.
6. Never create or switch branches; enforce this permanently in frontend and backend PROJECT_RULES.md.

## Compatibility notes

- Current backend `/api/auth/email-verification/resend`, `/verify`, `/onboarding/profile` and `/auth/login` already support the restored frontend steps. No backend schema was changed.
- Current backend registration contract stores email/password first, then resolves profile completion on verification. The previous frontend sign-up implementation supplied identity/account-intent values during registration that were not part of the backend input contract. Historical sign-up screen/contract is restored to match backend.
- Historical Candidate Chart dependencies `echarts ^5.6.0`, `echarts-for-react ^3.0.2` and lock graph restored without changing other existing package entries.
- This repair does not revert or turn off Recruitment → Candidates list, Add Candidate CV preview, or ESCO research. It restores missing UI rather than discarding unrelated work.

## Required verification

- GitHub frontend CI: typecheck and Vite build.
- Owner local: fresh `npm ci` after dependency changes and `npm run verify`; inspect:
  - PIN setup and unlock: obscured digits, numeric keypad, submission;
  - Sign In: link for missing verification email, resend with generic response;
  - registration → verify email → complete profile → company setup, including expired token;
  - Organization and Locations sidebar labels and routing;
  - System → Candidate Prototype: Resume tab, ECharts donut, actions;
  - Recruitment → Candidates remains present with Add Candidate.
- Do **not** claim backend email delivery (Cloudflare/local adapter), PIN login or ECharts runtime behavior verified until owner checks them locally.

## Non-goals

No permanent fine-grained permissions, Candidate database entities, new feature branches, force pushes, destructive resets, or unrelated refactors. This is a corrective restoration for diverged UI history.
