# System Administration UI

Last reviewed: 2026-10-01

This document describes platform-level SYSTEM_OWNER UI.

## Visibility
The System section is visible only when:
`session.access.platformRole === 'SYSTEM_OWNER'`.

## Navigation
System
- Control Panel
  - Overview
  - Organizations
  - Users
  - Requests

Current real route:
- `/system/organizations`.

Overview, Users and Requests remain placeholders.

## Organizations page
`/system/organizations` uses the approved Phoenix CRM Leads/table structure adapted to organizations and is backed by the real SYSTEM_OWNER organization activation API.

It currently includes:
- global search;
- creation-date filter;
- status filter modal;
- sorting;
- selection;
- pagination;
- row actions;
- localized labels.

## Organization activation queue
The screen loads through:
- `GET /api/system/organizations`.

The table shows:
- organization name;
- VAT number;
- PENDING/ACTIVE status;
- creator email;
- creator phone;
- legal-address country;
- creation date and time.

Pending organizations are surfaced first and can be activated from the row action menu.

The Control Panel new-item indicator is data-driven and is shown only while `pendingCount > 0`.

The page keeps the approved Phoenix CRM Leads table pattern.

Create Organization, City, INACTIVE status and Remove are not part of the current activation workflow. View remains visible but disabled until a review/details surface is implemented.

## Not yet complete
- System Overview;
- System Users;
- System Requests;
- organization review/details surface behind View.
