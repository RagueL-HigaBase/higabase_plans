<!-- Archived source from https://github.com/RagueL-HigaBase/higa_systems_react/blob/main/docs/production/SYSTEM_USER_PROFILE_EDIT.md; not current canonical state. -->

# Production Block — SystemUser Edit Profile

Status: **DONE**  
Frontend verified: **2026-09-27**

Canonical domain documents:
- `docs/SYSTEM_USER_PROFILE.md`
- `docs/PHOENIX_UI.md`

## Scope
This record closes the authenticated SystemUser Edit Profile UI and save behavior.

## Final frontend contract
The page uses the approved Phoenix two-column form layout.

Primary/left:
- Identity;
- Address;
- Social & Web;
- Profile Picture.

Right:
- Account;
- Personal;
- Contact;
- Change Password.

Other rules:
- Date of Birth uses the Phoenix EventsSchedule DatePicker render + Form.Floating pattern.
- Social links are LinkedIn, Facebook and X / Twitter.
- Account email/password remain a separate security mutation requiring current password.
- If account mutation succeeds first, session state is refreshed and password inputs are cleared before profile persistence continues.
- Complete successful Save Profile navigates to `/`.
- Profile Picture is preview-only until object storage is connected.
- No custom page CSS or third form-pattern family is used.
- Future forms must use the documented Primary Form Pattern or Secondary Form Pattern from Phoenix Create an Event.

## Verification
Verified locally by the project owner:
- `npm run verify`.

Observed result:
- TypeScript typecheck passed;
- production Vite build passed.

Non-blocking build warnings remain:
- third-party `lottie-web` eval warning;
- generated chunks above Vite default size threshold.

## Code-quality maintenance
On 2026-10-01, the Edit Profile submit flow was structurally refactored for SonarQube rule `typescript:S3776` (Cognitive Complexity). Account mutation, post-save session synchronization and save-error classification are separated into focused helpers.

The documented account/profile partial-save behavior is unchanged. The maintenance refactor was verified by the project owner on 2026-10-01 with a clean `npm run verify` run and a SonarQube re-scan showing 0 issues / 0 effort. It does not reopen the completed product behavior block.

## Reopen only if
Reopen this block if:
- Edit Profile fields/layout requirements change;
- account/profile save semantics change;
- object-storage photo persistence is implemented;
- the approved Phoenix form-pattern contract changes.

Otherwise treat frontend SystemUser Edit Profile as closed.
