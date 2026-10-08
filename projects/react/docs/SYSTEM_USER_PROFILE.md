# System User Profile UI

Last reviewed: 2026-09-27

This document describes the authenticated business user's personal Edit Profile flow. It is separate from Company Profile.

## Route
`/profile/edit`

## Phoenix structure
The page uses the approved two-column Phoenix form structure.

Left:
- Identity;
- Address;
- Social & Web;
- Profile Picture.

Right:
- Account;
- Personal;
- Contact;
- Change Password.

Phoenix structural rules are defined in `docs/PHOENIX_UI.md`.

## Fields
Identity:
- first name;
- last name;
- optional middle name.

Personal:
- gender;
- date of birth.

Contact:
- phone;
- optional WhatsApp.

Account:
- email.

Change Password:
- current password;
- new password;
- confirm new password.

Social & Web:
- optional LinkedIn URL;
- optional Facebook URL;
- optional X / Twitter URL.

Address:
- country;
- city;
- postal code;
- street;
- house number;
- optional additional field.

## Persistence
- `GET /api/profile` preloads values;
- `PUT /api/profile` saves personal/contact/address/social data transactionally;
- `PATCH /api/me/account` handles email/password separately with current-password verification.

## Validation
Frontend Zod mirrors backend validation.
Names use the approved English/ASCII spelling rule.
Address/business text follows the global ASCII-only rule.

## Account/profile partial save
If an account email/password change succeeds, the frontend refreshes session state and clears password inputs before continuing with personal-profile persistence. A later profile failure must not make the UI retry the already-saved account mutation with stale credentials.

## Photo
Current photo selection is preview-only.
Production object-storage upload is not connected yet.

## Localization
All visible Edit Profile strings and country/date localization use the existing 28-language system.
