# HigaBase UI Standards

Last reviewed: 2026-10-05

This document owns cross-project semantic UI conventions that should remain recognizable across HigaBase clients.

It defines semantic roles, not page-specific styling. Each client may map these roles to its native design system, but the hierarchy and intent must remain consistent.

## 1. Section label heading

Use for compact workspace labels and pane headings such as:
- Candidate name in the dossier identity block;
- System profile;
- Master of Occupation and Description;
- comparable compact domain/workspace labels.

Semantic rule:
- medium-emphasis heading;
- compact, not page-title scale;
- normal Higa/Phoenix heading family;
- do not replace with a custom font.

Web/Phoenix mapping:
- native Phoenix `h5` typography;
- Phoenix `text-body-emphasis` color where an explicit emphasis token is needed;
- no custom font-family, font-size or font-weight override.

This is the default heading level for this class of compact labels.

## 2. Supporting description

Use for secondary explanatory text directly associated with a compact heading or identity label, for example:
- registration date under a Candidate name;
- Candidate location/address summary;
- description under System profile;
- description under Master of Occupation and Description.

Semantic rule:
- 12.8px text size;
- muted secondary/tertiary body color;
- same visual tone across comparable surfaces;
- no custom color invention.

Web/Phoenix mapping:
- Phoenix `fs-9`;
- Phoenix `text-body-tertiary` for descriptive/supporting copy unless an existing Phoenix reference for the same semantic role requires `text-body-secondary`;
- preserve the theme color token instead of hardcoding a color value.

## 3. Consistency rule

Do not tune these roles independently per page.

When a new HigaBase surface needs:
- a compact domain/workspace label, use the Section label heading role;
- explanatory or contextual copy beneath it, use the Supporting description role.

Page layout, spacing, forms, navigation and component structure remain implementation/client concerns and are documented in the relevant repository.
