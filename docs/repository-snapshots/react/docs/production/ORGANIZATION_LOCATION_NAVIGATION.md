<!-- Archived source from https://github.com/RagueL-HigaBase/higa_systems_react/blob/main/docs/production/ORGANIZATION_LOCATION_NAVIGATION.md; not current canonical state. -->

# Organization Location Navigation

Verified: 2026-09-30

## Scope
Verified frontend navigation block for the new Organization Location architecture.

Implemented:
- Profile remains a direct Organization item.
- Main Location is added as a temporary frontend-only location item.
- Main Location uses the Phoenix Stock Details tab structure.
- Compact location header without breadcrumbs or duplicate large heading.
- Top tabs: Overview, Team, Operations, Finance, Compliance, Activity.
- Working submenu: Connections, Invoices, Team, Documents, Settings.
- Legacy standalone sidebar sections Operations, Finance, Compliance and Connections were removed together with their legacy standalone routes.
- Existing Team navigation remains.
- No backend/database Location persistence was added.

## Phoenix references
- `phoenix-react-reference/src/components/modules/stock/stock-details/StockDetailsMainContent.tsx`
- `phoenix-react-reference/src/components/navbars/navbar-vertical/NavbarVerticalMenu.tsx`

## Verification
Project owner verified locally on 2026-09-30:

```bash
npm run verify
```

TypeScript typecheck and Vite production build completed successfully.

Known non-blocking build warnings:
- third-party Lottie uses `eval`;
- generated chunks larger than 500 kB.
