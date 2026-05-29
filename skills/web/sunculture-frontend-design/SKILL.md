---
name: sunculture-frontend-design
description: >
  Apply SunCulture brand guidelines, design tokens, and UX patterns when
  building or reviewing web interfaces for SunCulture products (AMT frontend,
  Mopesa frontend, or any new SunCulture web surface). Covers the teal/green/
  yellow colour system and gradient palette, Circular Std typography, logo
  usage rules, semantic design tokens (@/styles/theme in AMT; TailwindCSS +
  shadcn/ui in Mopesa), SunCulture voice and copy rules (UK spelling, pragmatic
  idealism, benefit-led), and canonical UI patterns: forms (validate → confirm
  → mutate), confirmation dialogs (useConfirm), data tables (products.tsx
  pattern), detail pages with tabs, and permission-gated actions. Use when
  styling a component, reviewing code for brand alignment, writing UI copy, or
  building any new page or feature on a SunCulture product.
license: MIT
metadata:
  author: sunculture
  version: "1.0.0"
---

# SunCulture Frontend Design

> Source of truth for brand identity, design tokens, UI patterns, and web
> interface standards across all SunCulture web products.
>
> Brand PDF: `docs/SunCulture_Brand_Guidelines.pdf`
> AMT UX standards: `docs/frontend-ux-standards.md`
> AMT theme tokens: `src/styles/theme/`

---

## 1. Brand identity at a glance

**Brand promise:** Life-changing technology for everyday challenges.

**Brand personality:** Idealistic + Pragmatic (wonder anchored in facts).
Visionary + Collaborative (partner, not leader).

**Aesthetic direction:** Clean, optimistic, energetic. White backgrounds with
vivid teal/green/yellow accents and gradient moments. Not corporate, not stark
— warm and accessible with genuine confidence.

### Colour

| Token                     | Hex       | Use                                                                         |
| ------------------------- | --------- | --------------------------------------------------------------------------- |
| Interactive accent (teal) | `#07B282` | Primary actions, links, active states — `semanticColors.interaction.accent` |
| Brand teal                | `#00CCC8` | Brand headers, hero moments — `brandPrimitives.blue`                        |
| Yellow                    | `#F3D500` | Brand accent, gradient start — `brandPrimitives.yellow`                     |
| Green                     | `#62C628` | Success, positive states — `brandPrimitives.green`                          |
| Orange                    | `#FF8202` | Warning, energetic accent — `brandPrimitives.orange`                        |
| Red                       | `#E23D26` | Danger, error — `brandPrimitives.red`                                       |
| Charcoal                  | `#2F313A` | Body text, dark elements — `legacyDigital.charcoal`                         |
| Page background           | `#F7F8FA` | Page bg — `legacyDigital.pageBg` / `semanticColors.surface.page`            |

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

**System fallback:** Arial.

**Never use** Inter, Roboto, Space Grotesk, or system-ui as the primary font
on any SunCulture-branded surface.

| Token                 | Size | Use                           |
| --------------------- | ---- | ----------------------------- |
| `fontSize.micro`      | 10px | Fine print, badge counts      |
| `fontSize.caption`    | 11px | Labels, captions              |
| `fontSize.bodySmall`  | 12px | Filter labels, helper text    |
| `fontSize.body`       | 13px | Default body text             |
| `fontSize.bodyLarge`  | 14px | Card content, prominent body  |
| `fontSize.titleSmall` | 16px | Section headings, card titles |

### Logo rules (quick)

- Stacked primary logo by default; horizontal only when stacked won't fit.
- Min digital: 100px (stacked), 57px (horizontal).
- Clear space ≥ half the circle height on all sides.
- Never stretch, rotate, recolour, or place on an unsuitable background.

---

## 2. Design tokens by app

### AMT frontend (Next.js + Ant Design)

Always import from `@/styles/theme`. Never use raw hex values, raw px sizes,
or raw font sizes in JSX or inline styles.

```tsx
import {
  semanticColors,
  brandPrimitives,
  space,
  px,
  pad,
  inset,
  fontSize,
  fontWeight,
  lineHeight,
  radii,
  sizes,
  dataTableShellStyles,
  detailInfoCardStyles,
  navigationStyles,
  accountActionFormStyles,
} from "@/styles/theme";
```

**Colour hierarchy:**

- New UI → `semanticColors.*` (preferred)
- Intentional brand moments → `brandPrimitives.*`
- Legacy `legacyDigital.*` → migrate to semantic tokens when you touch the component

**Spacing:**

```tsx
space.xs; // 8    space.sm  // 12
space.md; // 16   space.lg  // 24   space.xl  // 32
px(space.md); // '16px'
pad(space.xs, space.md); // '8px 16px'
```

**Presets — use before rolling your own:**

| Preset                                    | Use for                                |
| ----------------------------------------- | -------------------------------------- |
| `dataTableShellStyles`                    | List pages with Card + filters + Table |
| `detailInfoCardStyles.shellWithBottomGap` | Hero card on detail pages              |
| `detailInfoCardStyles.shell`              | Tabs card on detail pages              |
| `navigationStyles`                        | Header, profile dropdown, breadcrumbs  |
| `accountActionFormStyles`                 | Action modals                          |

If a style repeats 2+ times in a feature, move it to a preset under
`src/styles/theme/presets/`.

### Mopesa frontend (Next.js + TailwindCSS + shadcn/ui)

Map brand colours to CSS variables in `app/globals.css`:

```css
:root {
  --sunculture-teal: #07b282;
  --sunculture-yellow: #f3d500;
  --sunculture-green: #62c628;
  --sunculture-orange: #ff8202;
}
```

Extend `tailwind.config.ts` with `colors: { 'sc-teal': 'var(--sunculture-teal)', ... }`.

Load Circular Std via `next/font/local` with Arial fallback.
All files must be kebab-case (Mopesa project convention).

---

## 3. Voice and copy rules

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
- **Button labels:** action verbs — "Create payplan", "Deactivate". Never "OK".
- **Date format:** `DD MMM YYYY HH:mm`. Guard with `moment(iso).isValid()`.
- **Empty cell value:** `'—'` (em dash). Never blank.
- **Loading states:** end with `…` — "Saving…", "Loading…"
- **Numbers:** numerals — "3 accounts", not "three accounts".

---

## 4. Core UI patterns

Full code shapes → `references/ui-patterns.md`

### Forms and modals

- `Form layout="vertical"` with `requiredMark`. Explicit `rules` on every field.
- **Custom footer** — never submit logic on Modal `onOk`.
- **Submit flow:** `validateFields()` → normalise → pre-checks → `useConfirm()` → mutate.
- Modal width: 520–560 for compact; 880 for multi-section.
- Always `destroyOnClose` when modal owns transient form state.
- Edit init: single `useEffect` on `[open, mode, record, form]`.

### Confirmation dialogs

Use `useConfirm()` from `@/components/confirm`:

```tsx
const confirmed = await confirm({
  title: "Save changes to this pay plan?",
  description: `You are about to update "${name}". Continue?`,
  confirmLabel: "Save changes",
  cancelLabel: "Back to form",
});
if (!confirmed) return;
```

- Required for all create / update / destructive / side-effect actions.
- `danger: true` for destructive actions.
- `cancelLabel: 'Back to form'` for form confirmations; `'Cancel'` for action buttons.
- **Never** use SweetAlert2 (`Swal.fire`) for new UI.

### Data tables

- `<Table bordered size="middle" />` + scoped CSS module border overrides.
- First column: `EyeOutlined` view link — `width: 48`, `fixed: 'left'`, `align: 'center'`, `aria-label`.
- `useTablePagination` for server-side pagination.
- Status: `<Tag>`. Booleans: `'Yes'`/`'No'`. Missing: `'—'`.
- Date columns: width 160, `moment()` formatted with `.isValid()` guard.
- Filter labels: `fontSize.bodySmall` `Typography.Text type="secondary"` above each filter.
- Any filter change resets pagination to page 1. No sticky columns.

### Detail pages

- Hero card (`detailInfoCardStyles.shellWithBottomGap`) + tabs card (`detailInfoCardStyles.shell`).
- Typed tab key allowlist; URL sync via `router.replace({...}, undefined, { shallow: true })`.
- Conditional tabs: reset to `overview` when active key is invalid for the current entity.
- Tab panel: dedicated component under `src/views/<domain>/`.
- Panel render order: loading placeholder → empty state → data.

### Permissions

- AMT: `useFeaturePermissions('feature_key')` → `canEdit` boolean.
- Mopesa: `<PermissionGate permission="expense.create">`.
- **Hide** (don't disable) actions when `!canEdit`.

---

## 5. Accessibility essentials

- `<button>` for actions, `<a>`/`<Link>` for navigation. Never `<div onClick>`.
- Icon-only buttons: `aria-label`. Decorative icons: `aria-hidden="true"`.
- Form fields: `<label>` (or `aria-label`) + explicit validation messages.
- Async updates: `aria-live="polite"`.
- Never `outline: none` without a `focus-visible` ring replacement.
- Ant Design icons on Next.js 15: add `onPointerEnterCapture={undefined} onPointerLeaveCapture={undefined}`.
- Images: `alt`, explicit `width` + `height`. Below-fold: `loading="lazy"`. Above-fold: `priority`.

---

## 6. Performance

- Virtualise lists > 50 items.
- `loading="lazy"` on below-fold images; `priority` on above-fold.
- Animate only `transform` / `opacity`. Never `transition: all`.
- Respect `prefers-reduced-motion`.
- URL reflects state: filters, tabs, pagination — use URL sync, not just local state.
- `font-display: swap`; preload critical fonts.

---

## 7. Anti-patterns

| Anti-pattern                             | Instead                                         |
| ---------------------------------------- | ----------------------------------------------- |
| Raw hex in JSX/CSS                       | `semanticColors.*` / `brandPrimitives.*` tokens |
| Raw `fontSize: 12`, raw `padding: '8px'` | `fontSize.*`, `space.*`, `px()`, `pad()`        |
| Inter / Roboto / Space Grotesk           | Circular Std / Arial                            |
| `Swal.fire(...)`                         | `useConfirm()`                                  |
| Modal `onOk` for form submit             | Custom footer pattern                           |
| Sticky table columns                     | `fixed: 'left'` on first column only            |
| "No data." empty state                   | Actionable empty state with context             |
| Silent async failures                    | `message.success()` + `message.error()`         |
| `[...map.values()]` spread               | `Array.from(map.values())`                      |
| `user-scalable=no` in viewport           | Remove — breaks a11y zoom                       |
| `onPaste` with `preventDefault`          | Never block paste                               |
| Hardcoded date formats                   | `moment()` / `Intl.DateTimeFormat`              |
| `outline: none` without focus ring       | Add `:focus-visible` ring                       |

---

## References

| Topic                                                                            | File                           |
| -------------------------------------------------------------------------------- | ------------------------------ |
| Full brand spec — colours, gradients, typography, logo, iconography, photography | `references/brand-identity.md` |
| AMT design tokens — full mapping, presets, CSS vars, Mopesa/Tailwind mapping     | `references/design-tokens.md`  |
| UI patterns — forms, modals, tables, detail pages, confirmation, permissions     | `references/ui-patterns.md`    |
| Web standards — accessibility, animation, performance, copy, i18n, dark mode     | `references/web-standards.md`  |
