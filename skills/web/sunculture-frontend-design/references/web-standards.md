# Web Interface Standards

Standards for all SunCulture web interfaces, derived from Vercel's Web
Interface Guidelines and extended with SunCulture-specific requirements.

---

## Mobile-first

Design and build for the smallest screen first, then progressively enhance
for larger viewports. Every standard below applies starting from mobile.

### Breakpoints

```css
/* base — mobile (320px+) */

@media (min-width: 768px) {
  /* tablet */
}
@media (min-width: 1280px) {
  /* desktop */
}
@media (min-width: 1536px) {
  /* wide */
}
```

### Touch

- Minimum tap target: **44×44px**. This applies to buttons, links, checkboxes,
  and any interactive element.
- `touch-action: manipulation` on interactive elements — eliminates the 300ms
  tap delay without disabling legitimate gestures.
- `-webkit-tap-highlight-color: transparent` on custom-styled elements to
  remove the default grey flash on tap (replace with a visible active state).
- `overscroll-behavior: contain` inside modals and drawers to prevent scroll
  chaining to the background page.

### Safe areas

Full-bleed layouts on iOS/Android require padding for notched devices:

```css
.page-shell {
  padding-bottom: env(safe-area-inset-bottom);
  padding-left: env(safe-area-inset-left);
  padding-right: env(safe-area-inset-right);
}
```

### Layout

- Avoid `overflow-x: hidden` on `body` / `html` — it breaks position:fixed elements.
  Apply it to the page wrapper instead.
- Prefer Flexbox / CSS Grid over JS measurement for responsive layout.
- Use `min-width: 0` on flex children that contain text — without it, truncation
  and wrapping will not work correctly.

---

## Semantic HTML

- Use `<button>` for actions, `<a>` for navigation.
  Never `<div onClick>` or `<span onClick>` — not keyboard accessible.
- Maintain hierarchical headings `<h1>` through `<h6>`. Don't skip levels.
  One `<h1>` per page.
- Include a skip-to-content link on complex layouts for keyboard navigation.
- Prioritise semantic HTML over ARIA roles. ARIA adds what HTML cannot express —
  don't duplicate roles the element already implies.

---

## Accessibility

### Labels

- Every form control needs a visible `<label>` (via `htmlFor` or wrapping the
  control) or an `aria-label` / `aria-labelledby`.
- Icon-only buttons: `aria-label` describing the action.
- Decorative icons: `aria-hidden="true"`.
- Async updates (toasts, inline validation): `aria-live="polite"`.

### Images

- Every `<img>` must have `alt`. Use `alt=""` for purely decorative images.
- Explicit `width` and `height` to prevent Cumulative Layout Shift (CLS).
- Below-fold: `loading="lazy"`. Above-fold critical (LCP): `fetchpriority="high"`.

### Focus states

- Every interactive element needs a visible `:focus-visible` indicator.
- Never `outline: none` or `outline: 0` without providing an explicit
  `:focus-visible` replacement — this fails WCAG 2.1 AA.
- Prefer `:focus-visible` over `:focus` to avoid showing rings on mouse click.
- Compound controls: use `:focus-within` to style the group when any part is focused.

```css
.btn:focus-visible {
  outline: 2px solid var(--color-focus-ring);
  outline-offset: 2px;
}
```

### Keyboard interaction

- Every mouse-operable element must be reachable by `Tab` and operable by
  `Enter` / `Space`.
- Modals and drawers: trap focus while open; restore focus to the trigger on close.
- `autoFocus`: use sparingly — desktop only, single obvious input, never on mobile
  (triggers viewport zoom on iOS).

### Colour contrast

- Body text on background: minimum **4.5:1** contrast ratio (WCAG AA).
- Large text (18px+ or 14px bold+): minimum **3:1**.
- Never communicate state with colour alone — always pair with text or an icon.
- Disabled controls: visually obvious. Where possible, **hide** rather than disable
  when the user lacks permission.

---

## Forms

- Inputs need `autocomplete` and a meaningful `name`.
- Use the correct `type` and `inputmode`:
  - `type="email"` + `inputmode="email"` for email fields
  - `type="tel"` + `inputmode="tel"` for phone numbers
  - `type="number"` or `inputmode="numeric"` for numeric fields
- Never block paste (`onPaste` with `preventDefault`).
- Labels must be clickable — use `htmlFor` matching `id`, or wrap the control.
- Disable spellcheck on codes, usernames, email fields.
- Submit button: enabled until request starts, then show a spinner. Re-enable on error.
- Inline errors: display next to the failing field; focus the first error on submit.
- Placeholder: example pattern ending with `…`, never a label substitute.
- Warn before navigation when the form has unsaved changes.

---

## Animation

```css
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

- Animate only `transform` and `opacity` — compositor-thread only, no layout triggers.
- Never `transition: all` — list specific properties explicitly.
- Set the correct `transform-origin` for scale and rotate animations.
- SVG animations: `transform-box: fill-box; transform-origin: center` on `<g>`.
- Make animations interruptible — don't lock UI state during a transition.
- One well-orchestrated page entrance beats scattered micro-interactions.

---

## Typography micro-rules

| Rule               | Correct                                   | Incorrect                    |
| ------------------ | ----------------------------------------- | ---------------------------- |
| Ellipsis           | `…` (U+2026)                              | `...`                        |
| Quotation marks    | `"text"` `'text'` (curly)                 | `"text"` `'text'` (straight) |
| Non-breaking space | `10&nbsp;MB`, `SunCulture&nbsp;RainMaker` | `10 MB`                      |
| Loading text       | "Saving…"                                 | "Saving..."                  |
| Number columns     | `font-variant-numeric: tabular-nums`      | Default                      |
| Heading wrap       | `text-wrap: balance` or `text-pretty`     | Default (widow words)        |

---

## Content handling

- Text containers must handle overflow: `overflow: hidden` + `text-overflow: ellipsis`,
  `line-clamp`, or `overflow-wrap: break-word` for unbreakable strings.
- Flex children containing text need `min-width: 0`.
- Anticipate very long inputs in every layout — test with 1-char and 200-char values.

---

## Performance

- Virtualise lists exceeding 50 items.
- Avoid layout reads in render (`getBoundingClientRect`, `offsetHeight`).
- Batch DOM reads and writes.
- Prefer uncontrolled inputs (`defaultValue`) where possible.
- Add `<link rel="preconnect">` for CDN and asset domains.
- Critical fonts: `<link rel="preload" as="font" crossorigin>` + `font-display: swap`.
- Use `sizes` on `<img>` / framework image components to serve the right resolution.

---

## Navigation and URL state

- URL must reflect UI state: active filters, active tab, pagination, open panels.
- Links use `<a>` — not `onClick` handlers — to support Cmd/Ctrl+click and middle-click.
- Deep-link all stateful UI.
- Destructive actions need confirmation or an undo window.

---

## Locale and dates

- Use `Intl.DateTimeFormat` for locale-aware date/time rendering.
- Use `Intl.NumberFormat` for numbers and currency.
- SunCulture display date format: `DD MMM YYYY HH:mm`. Always validate the input
  before formatting — invalid dates must render `'—'`, not `'Invalid Date'`.
- Wrap brand names and code tokens with `translate="no"` to prevent machine
  translation from mangling them.

---

## Hydration safety

- Guard date/time rendering against server/client mismatch — `new Date()` returns
  different values on server vs. client.
- Use `suppressHydrationWarning` only as a last resort; document why.
- Controlled inputs need `onChange` (or use `defaultValue` for uncontrolled).

---

## Dark mode

When a project supports dark/light mode:

- Set `color-scheme: dark` on `<html>` for dark themes (native scrollbars, form controls).
- Match `<meta name="theme-color">` to the current background.
- Explicit `background-color` and `color` on native `<select>` (not inherited in all browsers).
- Define all colours via CSS custom properties so toggling a class or attribute
  on `<html>` is sufficient to switch themes.

---

## Anti-pattern checklist

| Anti-pattern                             | Problem                              | Fix                            |
| ---------------------------------------- | ------------------------------------ | ------------------------------ |
| Desktop-first media queries              | Mobile experience often broken       | Mobile-first min-width queries |
| Touch targets under 44px                 | Unusable on mobile                   | Minimum 44×44px                |
| Hover-only affordances                   | Invisible on touch                   | Add tap/click equivalent       |
| `user-scalable=no` in viewport           | Breaks a11y zoom                     | Remove                         |
| `onPaste` with `preventDefault`          | Blocks paste, breaks a11y            | Remove                         |
| `transition: all`                        | Perf + unintended transitions        | List specific properties       |
| `outline: none` without `:focus-visible` | Fails keyboard a11y WCAG AA          | Add focus ring                 |
| `<div onClick>` / `<span onClick>`       | Not keyboard accessible              | Use `<button>` or `<a>`        |
| Images without `width` + `height`        | CLS                                  | Add dimensions                 |
| Large lists without virtualisation       | Performance                          | Virtualise                     |
| Form controls without labels             | Screen reader inaccessible           | Add `<label>`                  |
| Icon buttons without `aria-label`        | Screen reader inaccessible           | Add `aria-label`               |
| Hardcoded date formats                   | Locale issues / "Invalid Date" leaks | Validate + format via library  |
| Unjustified `autoFocus`                  | Disruptive on mobile (triggers zoom) | Desktop only, single input     |
| Empty state with no next action          | Unhelpful UX                         | Context + actionable link      |
| Silent async failure                     | User has no feedback                 | Always show success/error      |
