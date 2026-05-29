# SunCulture Design Tokens

Source: `src/styles/theme/` in the AMT frontend.

Always import from `@/styles/theme` — never from individual files inside the
directory. Never hardcode hex values, raw `px` sizes, or raw font sizes.

```tsx
import {
  semanticColors,
  brandPrimitives,
  brandGradientSolids,
  legacyDigital,
  antdTheme,
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
  payplansPageStyles,
  accountActionFormStyles,
} from "@/styles/theme";
```

---

## Colour tokens

### Semantic colours (`semanticColors`)

Use for all new UI. Semantic tokens communicate intent; they map to the correct
brand colour for the current context.

**Surface:**

| Token                             | Use                              |
| --------------------------------- | -------------------------------- |
| `semanticColors.surface.header`   | Application header background    |
| `semanticColors.surface.page`     | Main page background (`#F7F8FA`) |
| `semanticColors.surface.elevated` | Card and modal backgrounds       |
| `semanticColors.surface.dropdown` | Dropdown and popover backgrounds |

**Border:**

| Token                            | Use                              |
| -------------------------------- | -------------------------------- |
| `semanticColors.border.subtle`   | Lightest visible dividers        |
| `semanticColors.border.default`  | Standard border on inputs, cards |
| `semanticColors.border.hairline` | Thin 1px rule                    |
| `semanticColors.border.divider`  | Section separators               |

**Text:**

| Token                               | Use                         |
| ----------------------------------- | --------------------------- |
| `semanticColors.text.primary`       | Main body text              |
| `semanticColors.text.secondary`     | Supporting / secondary text |
| `semanticColors.text.iconMuted`     | Muted icon colour           |
| `semanticColors.text.heading`       | Page and card headings      |
| `semanticColors.text.bodySecondary` | Inline secondary body copy  |

**Semantic states:**

| Token                                  | Use                                           |
| -------------------------------------- | --------------------------------------------- |
| `semanticColors.semantic.danger`       | Error text, destructive labels                |
| `semanticColors.semantic.dangerStrong` | Error backgrounds, destructive action buttons |

**Interaction:**

| Token                                           | Use                                                   |
| ----------------------------------------------- | ----------------------------------------------------- |
| `semanticColors.interaction.accent`             | Primary actions, links, active indicators (`#07B082`) |
| `semanticColors.interaction.accentMutedSurface` | Hover / selected surface tint                         |
| `semanticColors.interaction.accentFocusRing`    | Focus ring colour                                     |

**Shadow:**

| Token                              | Use                         |
| ---------------------------------- | --------------------------- |
| `semanticColors.shadow.dropdown`   | Dropdown / popover shadow   |
| `semanticColors.shadow.cardLift`   | Hovered card elevation      |
| `semanticColors.shadow.tableShell` | Data table container shadow |

**Avatar:**

- `semanticColors.avatar.accentPalette` — array of background colours for
  user avatar initials.

---

### Brand primitives (`brandPrimitives`)

Use for intentional brand moments — gradient blocks, branded illustrations,
hero sections, marketing-aligned UI. Not for routine interactive UI.

```ts
brandPrimitives.yellow; // #F3D500
brandPrimitives.green; // #62C628
brandPrimitives.blue; // #00CCC8  (brand teal)
brandPrimitives.warmGrey; // #A0918A
brandPrimitives.orange; // #FF8202
brandPrimitives.red; // #E23D26
brandPrimitives.purple; // #9D2AAD
brandPrimitives.grey; // #636569
brandPrimitives.white; // #FFFFFF
```

---

### Gradient endpoint solids (`brandGradientSolids`)

Mid-point and end colours for the three canonical brand gradients.

```ts
brandGradientSolids.orange; // #FF5C00  (warm gradient: yellow → orange → red)
brandGradientSolids.red; // #DF180D
brandGradientSolids.limeGreen; // #55CC2E  (core brand: yellow → lime → blue)
brandGradientSolids.blue; // #00C7F7
brandGradientSolids.pink; // #E83A7B  (vibrant: yellow → pink → purple)
brandGradientSolids.purple; // #C800FF
```

**Constructing gradients in code:**

```ts
// Core brand gradient
`linear-gradient(135deg,
  ${brandPrimitives.yellow},
  ${brandGradientSolids.limeGreen},
  ${brandGradientSolids.blue})`
// Warm gradient
`linear-gradient(135deg,
  ${brandPrimitives.yellow},
  ${brandGradientSolids.orange},
  ${brandGradientSolids.red})`
// Vibrant gradient
`linear-gradient(135deg,
  ${brandPrimitives.yellow},
  ${brandGradientSolids.pink},
  ${brandGradientSolids.purple})`;
```

---

### Legacy digital (`legacyDigital`)

Read-only. For understanding existing component values. Migrate away from
`legacyDigital.*` to `semanticColors.*` when you touch a component.

```ts
legacyDigital.tealPrimary; // #07B282  (was the interactive accent)
legacyDigital.pageBg; // #F7F8FA
legacyDigital.charcoal; // #2F313A
legacyDigital.cyan; // #02C7F1
legacyDigital.navyDark; // #252543
legacyDigital.zinc; // #71717a
```

---

### Ant Design theme (`antdTheme`)

Passed once to `<ConfigProvider theme={antdTheme}>` at the app root.
Never nest multiple `ConfigProvider` components.

| Key            | Value     | Meaning                              |
| -------------- | --------- | ------------------------------------ |
| `colorPrimary` | `#07B082` | Teal — primary actions, focus, links |
| `colorSuccess` | `#62C628` | Green — success states               |
| `colorWarning` | `#FF8202` | Orange — warning states              |
| `colorError`   | `#DF180D` | Red — error states                   |

---

## Spacing (`space`)

```ts
space.xxs; // 4px
space.xs; // 8px
space.sm; // 12px
space.md; // 16px
space.lg; // 24px
space.xl; // 32px
space.gapInline; // 6px   — icon-to-text gap in buttons/labels
space.gapMenuIcon; // 10px  — menu item icon gap
```

### Spacing helpers

```ts
px(space.md); // '16px'
pad(space.xs, space.md); // '8px 16px'   (vertical, horizontal)
inset(8, 16, 8, 16); // '8px 16px 8px 16px'  (top, right, bottom, left)
```

---

## Typography

### Font size scale (`fontSize`)

All values are numbers (not strings). Pass to `fontSize: fontSize.body` in
style objects, or use with `px()` for CSS strings.

```ts
fontSize.micro; // 10  — fine print, badge counts
fontSize.caption; // 11  — labels, captions, timestamps
fontSize.bodySmall; // 12  — filter labels, helper text, secondary
fontSize.body; // 13  — default body text
fontSize.bodyLarge; // 14  — card content, prominent body
fontSize.titleSmall; // 16  — section headings, card titles
```

### Font weight (`fontWeight`)

```ts
fontWeight.normal; // 400 — Circular Std Book
fontWeight.medium; // 500 — Circular Std Medium
fontWeight.semibold; // 600 — labels, button text
fontWeight.bold; // 700 — Circular Std Bold
```

### Line height (`lineHeight`)

```ts
lineHeight.tight; // 1.2  — compact headings
lineHeight.snug; // 1.35 — subheadings
lineHeight.normal; // 1.5  — body text
```

---

## Border radius (`radii`)

```ts
radii.sm; // 6px  — small UI elements (tags, badges)
radii.md; // 8px  — inputs, buttons
radii.lg; // 12px — larger cards, panels
radii.cardShell; // 10px — card containers
```

---

## Sizes (`sizes`)

Fixed dimension values for common UI elements.

```ts
sizes.headerIcon; // 18px — navigation header icons
sizes.caretIcon; // 10px — dropdown caret icons
sizes.avatarHeader; // 26px — header avatar circle
sizes.avatarInitialsFont; // 11px — font size inside avatar
sizes.dropdownMinWidth; // 200px — minimum dropdown/select width
sizes.emailPreviewMaxWidth; // 120px — email preview truncation width
sizes.filterSelectSm; // 160px — small filter select width
sizes.filterSelectMd; // 192px — medium filter select width
sizes.filterSelectLg; // 224px — large filter select width
sizes.detailIntroMaxWidth; // 640px — max width for intro/hero text blocks
// Commission modals
sizes.commissionModalSm; // 560px
sizes.commissionModalMd; // 720px
sizes.commissionModalLg; // 920px
```

---

## Presets

Presets are style-object bundles for common layout patterns. Import and spread
them into component styles rather than rebuilding from scratch.

### `dataTableShellStyles` — list page shell

```tsx
<Card style={dataTableShellStyles.card}>
  <div style={dataTableShellStyles.filterRow}>
    <div>
      <Typography.Text
        type="secondary"
        style={dataTableShellStyles.filterLabel}
      >
        Search
      </Typography.Text>
      <Input.Search ... style={dataTableShellStyles.searchWrap} />
    </div>
  </div>
  <div style={dataTableShellStyles.tableWrap}>
    <Table ... />
  </div>
</Card>
```

| Key              | Purpose                                        |
| ---------------- | ---------------------------------------------- |
| `.card`          | Outer Card container (shadow, radius, padding) |
| `.filterRow`     | Horizontal filter controls layout              |
| `.filterLabel`   | 12px secondary label above each filter         |
| `.searchWrap`    | Search input sizing                            |
| `.tableWrap`     | Table container (overflow, padding)            |
| `.rowActionLink` | Inline row action link style                   |

### `detailInfoCardStyles` — detail page cards

```tsx
{/* Hero card — with margin below */}
<Card bordered={false} style={detailInfoCardStyles.shellWithBottomGap}>
  {/* title, eyebrow, action buttons */}
</Card>

{/* Tabs card — flush to bottom */}
<Card bordered={false} style={detailInfoCardStyles.shell}>
  <Tabs ... />
</Card>
```

### `navigationStyles` — app header

| Key               | Purpose                       |
| ----------------- | ----------------------------- |
| `.pageHeaderRow`  | Outer header row layout       |
| `.headerToolsRow` | Right-side tools area         |
| `.profileTrigger` | Avatar / profile click target |
| `.profileName`    | User name text in header      |
| `.profileEmail`   | User email text in header     |
| `.dropdownPanel`  | Profile dropdown container    |
| `.breadcrumb`     | Breadcrumb trail              |
| `.regionSwitcher` | Region selector button        |

### `payplansPageStyles` — pay plan list page

Pre-built layout for the pay plans index page structure. Extend or reference
when building similar product-domain list pages.

### `accountActionFormStyles` — action modals

Style preset for modal forms that perform account actions (status changes,
overrides, admin operations).

---

## CSS custom properties (`css-vars.css`)

`src/styles/theme/css-vars.css` mirrors semantic tokens as `:root` CSS
variables. Import in `_app.tsx` when Bootstrap components need access to
brand colours.

```css
:root {
  --color-accent: #07b082;
  --color-page-bg: #f7f8fa;
  --color-border-default: /* ... */;
  --color-text-primary: /* ... */;
}
```

---

## Mopesa / TailwindCSS + shadcn/ui mapping

Mopesa uses TailwindCSS 4.x + shadcn/ui. Brand colours live as CSS variables.

### `app/globals.css`

```css
:root {
  --sunculture-teal: #07b282;
  --sunculture-teal-light: #00ccc8;
  --sunculture-yellow: #f3d500;
  --sunculture-green: #62c628;
  --sunculture-orange: #ff8202;
  --sunculture-red: #e23d26;
  --sunculture-charcoal: #2f313a;
  --sunculture-page-bg: #f7f8fa;
}
```

### `tailwind.config.ts`

```ts
theme: {
  extend: {
    colors: {
      'sc-teal':    'var(--sunculture-teal)',
      'sc-yellow':  'var(--sunculture-yellow)',
      'sc-green':   'var(--sunculture-green)',
      'sc-orange':  'var(--sunculture-orange)',
      'sc-red':     'var(--sunculture-red)',
    },
  },
},
```

### Circular Std via `next/font/local`

```ts
import localFont from "next/font/local";

export const circularStd = localFont({
  src: [
    {
      path: "../assets/fonts/CircularStd-Book.woff2",
      weight: "400",
      style: "normal",
    },
    {
      path: "../assets/fonts/CircularStd-Medium.woff2",
      weight: "500",
      style: "normal",
    },
    {
      path: "../assets/fonts/CircularStd-Bold.woff2",
      weight: "700",
      style: "normal",
    },
  ],
  variable: "--font-circular",
  display: "swap",
  fallback: ["Arial", "sans-serif"],
});
```

Apply in `app/layout.tsx`:

```tsx
<html className={circularStd.variable}>
```

```css
/* globals.css */
body {
  font-family: var(--font-circular), Arial, sans-serif;
}
```
