# System User Profile

Last reviewed: 2026-09-27

This document describes the business user's personal profile. It is separate from the organization/company profile.

## Data model
`SystemUser`
- first name;
- optional middle name;
- last name;
- account email.

`SystemUserProfile`
- gender;
- date of birth;
- nullable future photo object-storage key.

`SystemUserContact`
- phone;
- optional WhatsApp number.

`SystemUserAddress`
- ISO alpha-2 country code;
- city;
- postal code;
- street;
- house number;
- optional additional address.

`SystemUserSocial`
- optional LinkedIn URL;
- optional Facebook URL;
- optional X / Twitter URL.

Email belongs only to SystemUser and is not duplicated in SystemUserContact.

## API
- `GET /api/profile`;
- `PUT /api/profile`.

PUT is one authenticated transactional profile update across SystemUser identity, SystemUserProfile, SystemUserContact, SystemUserAddress and SystemUserSocial.

## Validation
Names use English/ASCII passport spelling:
- A-Z / a-z;
- spaces;
- apostrophes;
- hyphens.

Other persisted business text follows the global ASCII-only invariant from PROJECT_RULES.md.

Phone fields use dedicated phone validation.

## Photo
The database stores only `hb_photo_object_key`.
Local-filesystem photo persistence is forbidden.
Production object-storage integration remains pending.


## Account credentials
Account email and password are not part of `PUT /api/profile`.
They are changed through `PATCH /api/me/account`, require the current password, and remain a separate security boundary from personal profile persistence.

The frontend must handle partial success explicitly: once account credentials are saved, it refreshes session/account state and clears password inputs before attempting the personal profile update.
