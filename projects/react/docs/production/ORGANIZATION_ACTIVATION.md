# Production Block — Organization Activation

Status: **DONE**  
Frontend verified: **2026-09-27**

Canonical domain document: `docs/SYSTEM_ADMIN.md`

## Scope
This record closes the SYSTEM_OWNER Organizations activation UI:
- real backend organization list;
- activation status display;
- pending-first queue;
- Activate row action;
- Control Panel pending indicator;
- search/date/status filtering.

## Final frontend contract
- The table uses the approved Phoenix CRM Leads pattern without custom CSS.
- Visible columns follow the Phoenix CRM Leads pattern: Organization, Email, Phone, Country, Create date/time and actions. VAT number and PENDING/ACTIVE badge are rendered on the secondary line inside the Organization cell.
- Status is PENDING or ACTIVE only.
- Demo organization data, separate VAT/Status columns, City, INACTIVE, Create Organization and Remove are not part of this workflow.
- Search is global and uses displayed organization data, including localized country names.
- DatePicker filters creation date.
- Filter modal contains All / Pending / Active only.
- PENDING rows expose Activate.
- View remains visible but disabled until a details/review surface is implemented.
- Control Panel blue indicator is shown only when pendingCount > 0.
- Successful activation refreshes shared organization state without a page reload.

## Verification
Verified locally by the project owner:
- `npm run verify`.

Observed result:
- TypeScript typecheck passed;
- production Vite build passed.

Non-blocking warnings remain:
- third-party `lottie-web` eval warning;
- generated chunks above Vite's default size warning threshold.

## Reopen only if
Reopen this block if:
- activation queue UI requirements change;
- backend activation-list contract changes;
- Control Panel pending indicator rules change;
- organization review/details is implemented.

Final visual refinement verified after closure: creator Email/Phone columns, VAT + status badge under Organization name, hidden status/VAT helpers for filtering/search, and locale labels behave correctly. Otherwise treat frontend organization activation as closed.
