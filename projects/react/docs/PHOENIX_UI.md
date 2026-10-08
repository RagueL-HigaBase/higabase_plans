# Phoenix UI Integration Rules

Last reviewed: 2026-10-02

Phoenix reference repository:
`RagueL-HigaBase/phoenix-react-reference`

Phoenix is read-only and is the visual/structural authority for approved UI patterns.

## Zero-invention rule
Before any Phoenix-derived UI change:
1. inspect the corresponding Phoenix source immediately before implementation;
2. verify component nesting, imports, React-Bootstrap primitives, classes, props, state behavior, assets and dependencies;
3. integrate the Phoenix structure 1:1;
4. adapt only business content, data, routing, validation and i18n unless the project owner explicitly approves a structural deviation;
5. do not invent custom wrappers, CSS, spacing, widths, breakpoint logic or replacement components;
6. if the reference cannot be located or is unclear, stop and inspect again instead of guessing;
7. compare the final implementation to the reference again before commit;
8. restore required Phoenix dependencies/assets instead of compensating with custom code.

No "close enough" implementations.

## Styling
Do not add custom CSS for approved Phoenix-derived surfaces unless explicitly requested by the project owner.

Use existing Phoenix:
- typography;
- classes;
- spacing;
- Bootstrap grid;
- form patterns;
- cards;
- modals;
- tables;
- navigation behavior.

## Business adaptation
Allowed examples:
- Event Title -> Company name;
- event content -> company-profile content;
- demo data -> API data;
- hardcoded text -> HigaBase i18n;
- event actions -> approved organization actions.

The adaptation must preserve the original Phoenix structural/layout pattern unless specifically approved otherwise.

## Verification
Never claim a Phoenix integration complete until:
- source-to-source comparison was performed;
- project owner reports `npm run verify` green.


## Approved form-pattern vocabulary

Canonical reference:
`phoenix-react-reference/src/pages/apps/events/CreateAnEvent.tsx`

Supporting reference components:
- `components/forms/EventDetailsForm.tsx`;
- `components/forms/EventsSchedule.tsx`;
- `components/forms/EventDescriptionForm.tsx`;
- `components/forms/EventTicketPricing.tsx`;
- `components/forms/EventCustomFields.tsx`.

### Primary Form Pattern
Source: the left/content column of Create an Event (`<Col xl={8}>` with `<Row className="gx-3 gy-4">`).

Use for:
- core entity/profile data;
- principal text/select/date fields;
- main descriptions;
- primary media/dropzone sections;
- primary multi-column field groups.

Structural rule:
- preserve Phoenix `Row/Col` grid and responsive widths;
- use Phoenix `FloatingLabel` / `Form.Floating` controls where the reference does;
- use the Phoenix DatePicker `render` pattern for floating date fields;
- preserve reference spacing utilities such as `gx-3`, `gy-4`, `mt-7`, `gy-6` only where the matching reference structure calls for them;
- no custom CSS or custom spacing abstraction.

### Secondary Form Pattern
Source: the right/settings column of Create an Event (`<Col xl={4}>`).

Use for:
- auxiliary settings;
- privacy/access choices;
- radio/checkbox groups;
- pricing/options;
- supporting/custom fields;
- compact configuration controls.

Structural rule:
- preserve Phoenix section separators and spacing such as `border-bottom border-translucent`, `pb-6`, `mb-6` where the matching reference uses them;
- use standard `Form.Group`, `Form.Label`, `Form.Control`, `Form.Select`, `Form.Check` and compact `Row/Col` structures as shown by the reference;
- do not convert Secondary controls into Primary floating controls unless explicitly approved.

### Selection rule
Every new form block must be classified as **Primary Form Pattern** or **Secondary Form Pattern** before implementation.

If the project owner says:
- “primary/main form” -> use Primary Form Pattern;
- “secondary/supporting form” -> use Secondary Form Pattern.

If the pattern is ambiguous, ask which of the two applies before implementing. Do not invent a third pattern or mix the two within one conceptual block without explicit approval.


## Higa information typography overlay

Higa information typography is an application-level semantic overlay on top of Phoenix. It does **not** replace Phoenix global typography, Bootstrap utilities, grid behavior, cards, spacing, or theme variables.

Canonical implementation:
`src/components/common/HigaInfoSurface.tsx`.

Use this component for compact label/value information blocks so the same visual hierarchy can be reused without copying page-specific CSS.

### A1 — information surface

A1 is the standard compact information pattern.

Shared typography:
- section heading: 20px / 600, Phoenix `text-body-emphasis`;
- label: uppercase, 12.8px / 700;
- value: 15px / 500;
- label/value grid: 1/3 + 2/3;
- label/value column gap: 2rem;
- row gap: 8px;
- solid FontAwesome icons;
- Phoenix `Badge` for semantic state values;
- missing values are omitted rather than rendered as placeholders.

Approved surfaces:
- **A1 · Page** — transparent page-background surface; no replacement global CSS;
- **A1 · Card** — the same typography inside a Phoenix Card.

A1 is the default compact information hierarchy, but page architecture remains entity-specific. Member Details may keep persistent context beside a workspace; Location Details uses its top tab bar as the primary workspace switch, with Overview content rendered inside the Overview tab. Do not force the Member Details two-column architecture onto Location Details.

### A2 — split information card

A2 uses the same information typography but places the heading and values in separate Phoenix card surfaces.

Approved surface:
- **A2 · Split card**.

Use A2 only where the split card structure is useful. Do not substitute A2 for A1 merely for decoration.

### Overlay rule

The Higa information layer must stay local and compositional:
- prefer `HigaInfoSurface` props/variants over global selector overrides;
- do not rewrite Phoenix heading, body, card, grid, or theme CSS;
- do not add page-specific duplicate typography rules when A1/A2 already fits;
- page layout remains Phoenix/Bootstrap-owned; Higa only supplies the approved semantic information hierarchy.
