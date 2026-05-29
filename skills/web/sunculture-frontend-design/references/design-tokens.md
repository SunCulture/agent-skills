# SunCulture Design Tokens

The SunCulture design token system translates brand values into reusable,
named constants. Implement these as CSS custom properties in your project.
Never hardcode raw hex values, raw `px` sizes, or raw font sizes — always
reference a token.

---

## Implementing in your project

Declare all tokens as CSS custom properties on `:root`. Your framework or
component library can then reference them consistently.

```css
/* globals.css / tokens.css */
:root {
  /* --- Colour: Interactive --- */
  --color-accent: #07b282; /* primary actions, links, active states */
  --color-accent-muted: #e6f7f2; /* hover / selected surface tint */
  --color-focus-ring: #07b282; /* keyboard focus ring */

  /* --- Colour: Semantic states --- */
  --color-success: #62c628;
  --color-warning: #ff8202;
  --color-danger: #e23d26;
  --color-danger-strong: #bf2e1a; /* danger backgrounds, destructive buttons */

  /* --- Colour: Brand primitives --- */
  --color-brand-teal: #00ccc8;
  --color-brand-yellow: #f3d500;
  --color-brand-green: #62c628;
  --color-brand-orange: #ff8202;
  --color-brand-red: #e23d26;
  --color-brand-purple: #9d2aad;
  --color-brand-warm-grey: #a0918a;
  --color-brand-grey: #636569;

  /* --- Colour: Surface --- */
  --color-page-bg: #f7f8fa;
  --color-surface-elevated: #ffffff;
  --color-surface-header: #ffffff;
  --color-surface-dropdown: #ffffff;

  /* --- Colour: Border --- */
  --color-border-subtle: rgba(0, 0, 0, 0.06);
  --color-border-default: rgba(0, 0, 0, 0.14);
  --color-border-hairline: rgba(0, 0, 0, 0.08);
  --color-border-divider: rgba(0, 0, 0, 0.1);

  /* --- Colour: Text --- */
  --color-text-primary: #2f313a;
  --color-text-secondary: #71717a;
  --color-text-muted: #a0918a;
  --color-text-heading: #2f313a;

  /* --- Colour: Shadow --- */
  --shadow-dropdown: 0 4px 12px rgba(0, 0, 0, 0.12);
  --shadow-card-lift: 0 2px 8px rgba(0, 0, 0, 0.08);
  --shadow-table: 0 1px 4px rgba(0, 0, 0, 0.06);

  /* --- Spacing --- */
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 12px;
  --space-4: 16px;
  --space-6: 24px;
  --space-8: 32px;
  --space-gap-inline: 6px; /* icon-to-text gap */
  --space-gap-menu: 10px; /* menu item icon gap */

  /* --- Typography --- */
  --font-family: "Circular Std", Arial, sans-serif;
  --font-size-xs: 10px; /* fine print, badge counts */
  --font-size-caption: 11px; /* labels, captions */
  --font-size-sm: 12px; /* filter labels, helper text */
  --font-size-base: 13px; /* default body text */
  --font-size-md: 14px; /* card content, prominent body — mobile minimum */
  --font-size-lg: 16px; /* section headings, card titles */

  --font-weight-normal: 400;
  --font-weight-medium: 500;
  --font-weight-semibold: 600;
  --font-weight-bold: 700;

  --line-height-tight: 1.2;
  --line-height-snug: 1.35;
  --line-height-normal: 1.5;

  /* --- Border radius --- */
  --radius-sm: 6px;
  --radius-md: 8px;
  --radius-lg: 12px;
  --radius-card: 10px;

  /* --- Component sizes --- */
  --size-touch-min: 44px; /* minimum tap target */
  --size-header-icon: 18px;
  --size-avatar: 26px;
  --size-dropdown-min-width: 200px;
  --size-detail-intro-max-width: 640px;
  --size-modal-sm: 520px;
  --size-modal-md: 720px;
  --size-modal-lg: 920px;
}
```

---

## Colour tokens in detail

### Interactive

| Token                  | Value     | Use                                             |
| ---------------------- | --------- | ----------------------------------------------- |
| `--color-accent`       | `#07B282` | Primary buttons, links, focus rings, active nav |
| `--color-accent-muted` | `#E6F7F2` | Hover / selected background tint                |
| `--color-focus-ring`   | `#07B282` | `:focus-visible` outline                        |

### Semantic states

| Token                   | Value     | Use                                    |
| ----------------------- | --------- | -------------------------------------- |
| `--color-success`       | `#62C628` | Success messages, positive badges      |
| `--color-warning`       | `#FF8202` | Warning messages, caution badges       |
| `--color-danger`        | `#E23D26` | Error text, destructive action labels  |
| `--color-danger-strong` | `#BF2E1A` | Error backgrounds, destructive buttons |

### Brand primitives

Use for gradient blocks, hero sections, and marketing moments — not routine
interactive UI.

| Token                     | Hex       |
| ------------------------- | --------- |
| `--color-brand-teal`      | `#00CCC8` |
| `--color-brand-yellow`    | `#F3D500` |
| `--color-brand-green`     | `#62C628` |
| `--color-brand-orange`    | `#FF8202` |
| `--color-brand-red`       | `#E23D26` |
| `--color-brand-purple`    | `#9D2AAD` |
| `--color-brand-warm-grey` | `#A0918A` |

---

## Gradients

Three canonical gradient sweeps. All start from yellow — the brand's unifying
warm energy point. Use as full-bleed colour blocks or multiply-blended over
photography. Never apply gradients to body text.

```css
.gradient-warm {
  background: linear-gradient(135deg, #f3d500, #ff5c00, #df180d);
}
.gradient-brand {
  background: linear-gradient(135deg, #f3d500, #55cc2e, #00c7f7);
}
.gradient-vibrant {
  background: linear-gradient(135deg, #f3d500, #e83a7b, #c800ff);
}
```

| Name       | From             | Via              | To               |
| ---------- | ---------------- | ---------------- | ---------------- |
| Warm       | Yellow `#F3D500` | Orange `#FF5C00` | Red `#DF180D`    |
| Core brand | Yellow `#F3D500` | Lime `#55CC2E`   | Blue `#00C7F7`   |
| Vibrant    | Yellow `#F3D500` | Pink `#E83A7B`   | Purple `#C800FF` |

---

## Spacing

Scale based on a 4px base unit. Use tokens consistently — avoid one-off values.

| Token       | Value | Common use                               |
| ----------- | ----- | ---------------------------------------- |
| `--space-1` | 4px   | Icon gap, tight internal spacing         |
| `--space-2` | 8px   | Small padding, gap between related items |
| `--space-3` | 12px  | Input padding, compact card spacing      |
| `--space-4` | 16px  | Default section padding, card inner      |
| `--space-6` | 24px  | Between sections, generous card padding  |
| `--space-8` | 32px  | Page-level breathing room                |

---

## Typography

### Circular Std loading

Load via your framework's font loading mechanism. Always provide Arial as
fallback to maintain brand character when the custom font is unavailable.

```css
@font-face {
  font-family: "Circular Std";
  src: url("/fonts/CircularStd-Book.woff2") format("woff2");
  font-weight: 400;
  font-display: swap;
}
@font-face {
  font-family: "Circular Std";
  src: url("/fonts/CircularStd-Medium.woff2") format("woff2");
  font-weight: 500;
  font-display: swap;
}
@font-face {
  font-family: "Circular Std";
  src: url("/fonts/CircularStd-Bold.woff2") format("woff2");
  font-weight: 700;
  font-display: swap;
}

body {
  font-family: var(--font-family);
  font-size: var(--font-size-base);
  line-height: var(--line-height-normal);
  color: var(--color-text-primary);
}
```

### Size scale

| Token                 | Value | Use                                           |
| --------------------- | ----- | --------------------------------------------- |
| `--font-size-xs`      | 10px  | Fine print, badge counts                      |
| `--font-size-caption` | 11px  | Labels, timestamps, captions                  |
| `--font-size-sm`      | 12px  | Filter labels, helper text, secondary UI      |
| `--font-size-base`    | 13px  | Default body text                             |
| `--font-size-md`      | 14px  | Card content, prominent body — mobile minimum |
| `--font-size-lg`      | 16px  | Section headings, card titles                 |

**Mobile floor:** never render text below `14px` on mobile viewports.

### Weight usage

| Weight   | Value | Use                            |
| -------- | ----- | ------------------------------ |
| Normal   | 400   | Body copy, field values        |
| Medium   | 500   | Subheadings, navigation labels |
| Semibold | 600   | Button text, table headers     |
| Bold     | 700   | Display headings, hero copy    |

---

## Border radius

| Token           | Value | Use                          |
| --------------- | ----- | ---------------------------- |
| `--radius-sm`   | 6px   | Tags, badges, small chips    |
| `--radius-md`   | 8px   | Inputs, buttons              |
| `--radius-lg`   | 12px  | Larger panels, sheet headers |
| `--radius-card` | 10px  | Card containers              |

---

## Component sizes

| Token                           | Value | Use                             |
| ------------------------------- | ----- | ------------------------------- |
| `--size-touch-min`              | 44px  | Minimum tap target height/width |
| `--size-header-icon`            | 18px  | Navigation icons                |
| `--size-avatar`                 | 26px  | Header user avatar              |
| `--size-dropdown-min-width`     | 200px | Select / dropdown minimum width |
| `--size-detail-intro-max-width` | 640px | Hero/intro text block max-width |
| `--size-modal-sm`               | 520px | Compact form dialogs            |
| `--size-modal-md`               | 720px | Standard multi-field dialogs    |
| `--size-modal-lg`               | 920px | Multi-section / wide dialogs    |

---

## Common layout patterns

Rather than library-specific presets, implement these as reusable CSS classes
or framework components in your project.

### Page shell

```css
.page-shell {
  background: var(--color-page-bg);
  min-height: 100vh;
  padding: var(--space-4);
}
@media (min-width: 768px) {
  .page-shell {
    padding: var(--space-6);
  }
}
```

### Card

```css
.card {
  background: var(--color-surface-elevated);
  border-radius: var(--radius-card);
  box-shadow: var(--shadow-card-lift);
  padding: var(--space-4);
}
@media (min-width: 768px) {
  .card {
    padding: var(--space-6);
  }
}
```

### Filter row (data table shell)

```css
.filter-row {
  display: flex;
  flex-direction: column;
  gap: var(--space-3);
  margin-bottom: var(--space-4);
}
@media (min-width: 768px) {
  .filter-row {
    flex-direction: row;
    align-items: flex-end;
  }
}

.filter-label {
  display: block;
  font-size: var(--font-size-sm);
  color: var(--color-text-secondary);
  margin-bottom: var(--space-1);
}
```
