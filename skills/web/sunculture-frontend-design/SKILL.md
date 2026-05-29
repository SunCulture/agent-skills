---
name: sunculture-frontend-design
description: >
  Apply SunCulture brand guidelines, design system, and UX patterns when
  building web interfaces. Covers the teal/green/yellow colour system and
  gradient palette, Circular Std typography, logo rules, the full design token
  system (colours, spacing, typography, radius), voice and copy rules (UK
  spelling, pragmatic idealism, benefit-led), a mobile-first layout approach,
  and core UI patterns: forms (validate → confirm → mutate), confirmation
  dialogs, data tables, and detail pages with tabs. Also enforces web interface
  standards for accessibility, focus states, animation, and performance. Use
  when styling a component, reviewing UI for brand alignment, writing UI copy,
  or building any new SunCulture web interface — regardless of framework.
license: MIT
metadata:
  author: sunculture
  version: "1.1.0"
---

# SunCulture Frontend Design

> Source of truth for brand identity, design tokens, UI patterns, and web
> interface standards across all SunCulture web products.
>
> Full brand spec: `references/brand-identity.md`
> Design token values: `references/design-tokens.md`

---

## 1. Mobile-first approach

Design and build for the smallest screen first, then progressively enhance
for larger viewports. This is a hard requirement — not a preference.

### Core rule

Write your base styles for mobile. Use `min-width` media queries to add
complexity at larger sizes. Never start from desktop and subtract.

```css
/* base — mobile */
.card {
  padding: 16px;
  flex-direction: column;
}

/* tablet and up */
@media (min-width: 768px) {
  .card {
    flex-direction: row;
  }
}

/* desktop and up */
@media (min-width: 1280px) {
  .card {
    padding: 24px;
  }
}
```

### Breakpoints

| Name    | Min-width | Target                    |
| ------- | --------- | ------------------------- |
| mobile  | —         | Default, 320px–767px      |
| tablet  | `768px`   | iPad, small laptop        |
| desktop | `1280px`  | Standard laptop/desktop   |
| wide    | `1536px`  | Large monitor / dashboard |

### Touch and interaction

- Minimum touch target size: **44×44px** for any tappable element.
- Use `touch-action: manipulation` to remove the 300ms tap delay.
- Place primary actions in the **lower half** of the screen (thumb-reachable).
- Avoid hover-only affordances — mobile has no hover state.
- Test navigation and modals with a real mobile device or emulator before shipping.

### Layout

- Stack content vertically on mobile. Use grid/flex rows only from `tablet` up.
- Fluid widths (`%`, `fr`, `min()`) over fixed `px` widths where possible.
- Minimum readable font size: **14px** on mobile (never scale below this).
- Generous line-height (`1.5`) on body text for mobile readability.
- Avoid horizontal scrollbars — `overflow-x: hidden` on the page root.
- Full-bleed sections on mobile; add max-width containers from `tablet` up.

### Forms on mobile

- Full-width inputs on mobile (`width: 100%`).
- Stack label above input — never inline at small sizes.
- Use the correct `type` and `inputmode` to trigger the right soft keyboard.
- Confirm dialogs should be full-screen sheets or bottom drawers on mobile,
  not centred desktop-style modals.

---

## 2. Brand identity at a glance

**Brand promise:** Life-changing technology for everyday challenges.

**Brand personality:** Idealistic + Pragmatic (wonder anchored in facts).
Visionary + Collaborative (partner, not leader).

**Aesthetic direction:** Clean, optimistic, energetic. White backgrounds with
vivid teal/green/yellow accents and gradient moments. Warm and accessible with
genuine confidence — not corporate, not stark.

### Colour

| Role               | Hex       | Use                                   |
| ------------------ | --------- | ------------------------------------- |
| Interactive accent | `#07B282` | Primary actions, links, active states |
| Brand teal         | `#00CCC8` | Brand headers, hero moments           |
| Yellow             | `#F3D500` | Brand accent, gradient start          |
| Green              | `#62C628` | Success, positive states              |
| Orange             | `#FF8202` | Warning, energetic accent             |
| Red                | `#E23D26` | Danger, error                         |
| Charcoal           | `#2F313A` | Body text, dark elements              |
| Page background    | `#F7F8FA` | Page / surface background             |

**Gradients** — three canonical sweeps. Use as solid colour blocks or
multiply-blended over hero images. Never on body text.

| Name       | Stops                                                  |
| ---------- | ------------------------------------------------------ |
| Warm       | `#F3D500` → `#FF5C00` → `#DF180D` (yellow–orange–red)  |
| Core brand | `#F3D500` → `#55CC2E` → `#00C7F7` (yellow–lime–blue)   |
| Vibrant    | `#F3D500` → `#E83A7B` → `#C800FF` (yellow–pink–purple) |

### Typography

**Primary font:** Circular Std — Book (400), Medium (500), Bold (700).
Round letterforms complement the circle logo. Friendly, clean, professional.
**Fallback:** Arial.

**Never use** Inter, Roboto, Space Grotesk, or system-ui as the primary font.

| Scale       | Size | Use                                           |
| ----------- | ---- | --------------------------------------------- |
| Micro       | 10px | Fine print, badge counts                      |
| Caption     | 11px | Labels, captions                              |
| Body small  | 12px | Filter labels, helper text                    |
| Body        | 13px | Default body text                             |
| Body large  | 14px | Card content, prominent body — mobile minimum |
| Title small | 16px | Section headings, card titles                 |

### Logo rules (quick)

- Stacked primary logo by default; horizontal only when stacked won't fit.
- Min digital: 100px (stacked), 57px (horizontal).
- Clear space ≥ half the circle height on all sides.
- Never stretch, rotate, recolour, or place on an unsuitable background.

---

## 3. Design tokens

The SunCulture design system is defined as a set of named token values.
Implement them as CSS custom properties in your project and reference them
throughout your styles. Never hardcode raw hex values, raw `px` sizes, or raw
font sizes — always use a token.

```css
:root {
  /* Colour — interactive */
  --color-accent: #07b282;
  --color-accent-muted: #e6f7f2;
  --color-focus-ring: #07b282;

  /* Colour — semantic */
  --color-success: #62c628;
  --color-warning: #ff8202;
  --color-danger: #e23d26;

  /* Colour — brand primitives */
  --color-brand-teal: #00ccc8;
  --color-brand-yellow: #f3d500;
  --color-brand-green: #62c628;
  --color-brand-orange: #ff8202;
  --color-brand-red: #e23d26;

  /* Colour — surface */
  --color-page-bg: #f7f8fa;
  --color-surface-elevated: #ffffff;
  --color-text-primary: #2f313a;
  --color-text-secondary: #71717a;
  --color-border-default: rgba(0, 0, 0, 0.14);

  /* Spacing */
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 12px;
  --space-4: 16px;
  --space-6: 24px;
  --space-8: 32px;

  /* Typography */
  --font-family: "Circular Std", Arial, sans-serif;
  --font-size-xs: 10px;
  --font-size-sm: 12px;
  --font-size-base: 13px;
  --font-size-md: 14px;
  --font-size-lg: 16px;

  /* Radius */
  --radius-sm: 6px;
  --radius-md: 8px;
  --radius-lg: 12px;
  --radius-card: 10px;
}
```

Full token reference (all values, gradient construction, spacing helpers):
`references/design-tokens.md`

**Colour hierarchy in practice:**

1. Semantic/functional tokens (`--color-accent`, `--color-danger`) for UI states
2. Brand primitives (`--color-brand-*`) for intentional brand moments and gradients
3. Surface and text tokens for backgrounds, borders, and copy

---

## 4. Voice and copy rules

SunCulture uses **UK spelling**. Tone: benefit-led, grounded, human. Never corporate.

| Rule              | Do                                  | Don't                          |
| ----------------- | ----------------------------------- | ------------------------------ |
| Short words       | "Help"                              | "Assist"                       |
| Contractions      | "It's", "Let's"                     | "It is", "Let us"              |
| Second person     | "You can take control"              | "Users can take control"       |
| Benefit-led       | "Triple your yields"                | "Powerful yield management"    |
| Anchor aspiration | "…saving up to $1,500 over 3 years" | "…changing lives" (unanchored) |
| UK spelling       | "Optimise", "Colour"                | "Optimize", "Color"            |
| Active voice      | "Save changes?"                     | "Changes will be saved?"       |

**UI copy checklist:**

- **Empty states:** name what is missing AND offer the next action. Never "No data."
- **Errors:** explain what went wrong + what to do next.
- **Confirmation title:** clear yes/no question — "Save changes to this pay plan?"
- **Confirmation description:** name the entity + most consequential values.
- **Button labels:** action verbs — "Create", "Save changes", "Deactivate". Never "OK".
- **Date format:** `DD MMM YYYY HH:mm`. Always validate before formatting.
- **Empty cell value:** `'—'` (em dash). Never blank.
- **Loading states:** end with `…` — "Saving…", "Loading…"
- **Numbers:** numerals — "3 accounts", not "three accounts".

---

## 5. Core UI patterns

Full framework-agnostic code shapes → `references/ui-patterns.md`

### Forms

- Labels above fields — never inline at any viewport.
- Explicit validation message on every required field — never rely on browser/library defaults.
- Full-width inputs on mobile; constrain width only from `tablet` up.
- **Submit flow:** validate fields → normalise → optional pre-checks → confirm → mutate.
- Never submit on Enter without explicit confirmation for destructive or irreversible actions.
- Edit-mode init: pre-fill all fields before the form becomes visible.

### Confirmation dialogs

Every create / update / destructive / side-effect action requires confirmation.

- **Title:** a clear yes/no question.
- **Description:** names the entity AND the most consequential values being changed.
- **Confirm label:** action verb — "Create", "Save changes", "Deactivate". Never "OK".
- **Cancel label:** "Back to form" when triggered from a form; "Cancel" for action buttons.
- Destructive actions: visually distinct (danger colour on confirm button).
- On mobile: use a bottom sheet or full-screen dialog, not a centred modal.

### Data tables

- Server-side pagination — never load all rows at once.
- First column: icon-only view link with `aria-label`.
- Status: badge/tag (success / neutral). Booleans: `'Yes'` / `'No'`. Missing values: `'—'`.
- Date columns: consistent `DD MMM YYYY HH:mm` format. Validate before formatting.
- Filter labels visible above each filter control.
- Any filter change resets pagination to page 1.
- On mobile: consider a card list instead of a scrolling table.

### Detail pages

- Hero section + tabbed content panels.
- Typed tab key allowlist — validate URL param before applying.
- URL reflects active tab (deep-linkable).
- Conditional tabs: reset to default tab when the active key is no longer valid.
- Tab panel render order: loading → empty state (actionable) → data.

### Permissions

- **Hide** (don't disable) actions the user cannot perform.
- The detail page (read view) stays accessible to read-only users.
- Always check permissions server-side — client-side hiding is UX only.

---

## 6. Accessibility essentials

- `<button>` for actions, `<a>` for navigation. Never `<div onClick>`.
- Icon-only buttons: `aria-label`. Decorative icons: `aria-hidden="true"`.
- Form fields: visible `<label>` + explicit error messages.
- Async updates (toasts, validation): `aria-live="polite"`.
- Never `outline: none` without a `:focus-visible` ring replacement.
- Images: `alt`, explicit `width` + `height`.
  Below-fold: `loading="lazy"`. Above-fold: `fetchpriority="high"`.
- Don't communicate state with colour alone — pair with text or icon.

---

## 7. Performance

- Virtualise lists > 50 items.
- Animate only `transform` / `opacity`. Never `transition: all`.
- Respect `prefers-reduced-motion`.
- URL reflects state: filters, tabs, pagination — sync to URL, not just local state.
- `font-display: swap`; preload critical fonts.
- Optimise images: explicit dimensions, `loading="lazy"` below the fold, correct `sizes`.

---

## 8. Anti-patterns

| Anti-pattern                              | Instead                                            |
| ----------------------------------------- | -------------------------------------------------- |
| Raw hex colours in styles                 | Design token CSS variables                         |
| Raw `font-size: 12px`, raw padding        | Token variables (`--font-size-sm`, `--space-3`)    |
| Inter / Roboto / Space Grotesk            | Circular Std / Arial                               |
| Desktop-first layout                      | Mobile-first with `min-width` queries              |
| Touch targets under 44px                  | Minimum 44×44px                                    |
| Hover-only affordances                    | Always provide a visible/tap-accessible equivalent |
| Confirming nothing on destructive actions | Confirmation dialog, always                        |
| Sticky table columns beyond first         | Use first column only                              |
| "No data." empty state                    | Actionable empty state with context                |
| Silent async failures                     | Always surface success / error feedback            |
| `user-scalable=no` in viewport            | Remove — breaks accessibility zoom                 |
| Blocking paste (`onPaste preventDefault`) | Never block paste                                  |
| Hardcoded date format strings             | Use a date library with locale support             |
| `outline: none` without focus ring        | Add `:focus-visible` ring                          |

---

## References

| Topic                                                                             | File                           |
| --------------------------------------------------------------------------------- | ------------------------------ |
| Full brand spec — colours, gradients, typography, logo, iconography, photography  | `references/brand-identity.md` |
| Design token values — colours, spacing, typography, radius + implementation guide | `references/design-tokens.md`  |
| UI patterns — forms, confirmation, data tables, detail pages, permissions         | `references/ui-patterns.md`    |
| Web standards — accessibility, animation, performance, copy, i18n, mobile         | `references/web-standards.md`  |
