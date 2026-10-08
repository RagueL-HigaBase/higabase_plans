# System Administration

Last reviewed: 2026-10-03

This document describes platform-level administration and SYSTEM_OWNER behavior.

## Platform roles
- VIEWER;
- ADMIN;
- SYSTEM_OWNER.

Platform roles are separate from organization roles.

There may be only one SYSTEM_OWNER.

## SYSTEM_OWNER bootstrap
Runtime configuration:
- `SYSTEM_OWNER_EMAIL`;
- `SYSTEM_OWNER_FIRST_NAME`;
- `SYSTEM_OWNER_LAST_NAME`;
- `SYSTEM_OWNER_ORGANIZATION_NAME` (default `HigaBase`).

Startup guarantees:
- configured email matches the single SYSTEM_OWNER;
- missing SYSTEM_OWNER is created with an unknown cryptographically random password;
- the configured SYSTEM_OWNER email is trusted deployment configuration and is marked verified at bootstrap;
- an existing normal account is never silently promoted;
- a conflicting different SYSTEM_OWNER causes startup failure;
- an already-configured SYSTEM_OWNER keeps existing password/session state.

The bootstrap password is never logged or stored in plaintext.

First access uses the normal recovery path:
`Forgot Password → real email → Reset Password → Sign In`.

SYSTEM_OWNER password-reset email uses the same shared EmailProvider as other transactional auth mail.

## HigaBase organization ownership
SYSTEM_OWNER also participates in normal organization RBAC for HigaBase's own company-administration surface.

Startup ensures the SYSTEM_OWNER has one OWNER organization membership:
- existing OWNER membership is preserved;
- if none exists, an active organization is created using SYSTEM_OWNER_ORGANIZATION_NAME;
- no duplicate organization is created once ownership exists.

This keeps company administration on the same ADMIN/OWNER organization APIs instead of introducing a SYSTEM_OWNER bypass.

## Organization activation
SYSTEM_OWNER is the only platform role currently allowed to activate newly created organizations.

Activation is transactional:
- organization activation timestamp and activating SYSTEM_OWNER are stored;
- the original organization creator receives OWNER membership;
- repeated activation returns the already-active result.

## SYSTEM_OWNER organization list
`GET /api/system/organizations` returns the real organization activation queue for SYSTEM_OWNER only.

The list exposes:
- organization id and name;
- VAT number;
- legal-address country code;
- status derived as PENDING or ACTIVE from `hb_activated_at`;
- creation timestamp;
- pending count.

Default ordering is:
1. PENDING organizations first;
2. oldest PENDING first;
3. ACTIVE organizations afterwards;
4. newest ACTIVE first.

There is no separate INACTIVE activation status in the persistence model.
