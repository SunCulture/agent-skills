# Web Interface Standards

Standards for all SunCulture web interfaces. Derived from Vercel's Web
Interface Guidelines, extended with SunCulture-specific stack guidance.

---

## Semantic HTML

- Use `<button>` for actions, `<a>` / `<Link>` for navigation.
  Never `<div onClick>` or `<span onClick>` — they are not keyboard accessible.
- Maintain hierarchical headings `<h1>` through `<h6>`. Don't skip levels.
- One `<h1>` per page.
- Include a skip-to-content link on complex layouts for keyboard navigation.
- Prioritise semantic HTML over ARIA roles. ARIA adds what HTML can't express —
  don't duplicate roles already implied by the element.

---

## Accessibility

### Labels

- Every form control needs a visible `<label>` (via `htmlFor` or wrapping the
  control) or an `aria-label` / `aria-labelledby`.
- Icon-only buttons need `aria-label` describing the action.
- Decorative icons: `aria-hidden="true"` so screen readers skip them.
- Async updates (toasts, inline validation): `aria-live="polite"`.

### Ant Design icon workaround (Next.js 15 + React 18)

TypeScript raises errors on `@ant-design/icons` elements in this environment.
Add to every icon usage:

```tsx
<EyeOutlined
  aria-hidden
  onPointerEnterCapture={undefined}
  onPointerLeaveCapture={undefined}
/>
```

### Images

- Every `<img>` must have `alt`. Use `alt=""` for purely decorative images.
- Provide explicit `width` and `height` attributes to prevent Cumulative Layout
  Shift (CLS).
- Below-fold images: `loading="lazy"`.
- Above-fold critical images (LCP): `priority` (Next.js `<Image>`) or
  `fetchpriority="high"` on native `<img>`.

### Focus states

- Every interactive element needs a visible focus indicator.
  Use `focus-visible:ring-*` (Tailwind) or a `:focus-visible` CSS rule.
- Never `outline: none` or `outline: 0` without providing an explicit
  `:focus-visible` replacement — this fails WCAG 2.1 AA.
- Prefer `:focus-visible` over `:focus` to avoid showing rings on mouse click.
- Use `:focus-within` on compound controls (e.g. a search input with a button)
  to style the whole group when any part is focused.

### Keyboard interaction

- Every interactive element reachable by mouse must also be reachable by
  `Tab` key and operable by `Enter` / `Space`.
- Modal and drawer components: trap focus while open; restore focus to the
  trigger element on close.
- `autoFocus`: use sparingly — desktop only, single obvious input field.
  Never on mobile (triggers viewport zoom on iOS).

### Colour contrast

- Text on backgrounds: minimum 4.5:1 contrast ratio (WCAG AA).
- Large text (18px+ or 14px bold+): minimum 3:1.
- Don't communicate state with colour alone — always pair with text or an icon.
- Disabled controls should be visually obvious; where possible, **hide** rather
  than disable when the user lacks permission.

---

## Forms

- Inputs need `autocomplete` and a meaningful `name` attribute.
- Use the correct `type` and `inputmode`:
  - `type="email"` + `inputmode="email"` for email fields.
  - `type="tel"` + `inputmode="tel"` for phone numbers.
  - `type="number"` or `inputmode="numeric"` for numeric fields.
- Never block paste functionality (`onPaste` with `preventDefault` is forbidden).
- Labels must be clickable — use `htmlFor` matching the input `id`, or wrap
  the control inside `<label>`.
- Disable spellcheck on codes, usernames, and email fields:
  `spellCheck={false}` / `autoCorrect="off"` / `autoCapitalize="off"`.
- Submit button: enabled until the request starts, then show a spinner.
  Re-enable on error so the user can retry.
- Display validation errors inline, next to the failing field.
  On form submit, focus the first errored field.
- Placeholders show an example pattern ending with `…`, not a label.
  (`placeholder="e.g. CSD1+RM2S MAX…"` not `placeholder="Enter name"`).
- Warn before navigation (browser `beforeunload` or router guard) when the
  form has unsaved changes.

---

## Animation

```css
/* Always respect reduced-motion */
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

- Animate only `transform` and `opacity` — these run on the compositor thread
  and don't trigger layout.
- Never `transition: all` — list specific properties to avoid unintended
  transitions and improve performance.
- Set the correct `transform-origin` for scale and rotate animations.
- SVG animations: set `transform-box: fill-box; transform-origin: center` on
  the `<g>` element.
- Make animations interruptible — don't lock UI state during a transition.
- One well-orchestrated entrance animation beats scattered micro-interactions.
  Consider page-load staggered reveals for high-impact moments.

**React animations:**

- Use the Motion library (`motion/react`) for complex sequences and gesture-
  based interactions.
- For simple enter/exit: CSS transitions are preferred (no JS bundle cost).

---

## Typography micro-rules

| Rule               | Correct                               | Incorrect                    |
| ------------------ | ------------------------------------- | ---------------------------- |
| Ellipsis character | `…` (U+2026)                          | `...` (three dots)           |
| Quotation marks    | `"text"` `'text'` (curly)             | `"text"` `'text'` (straight) |
| Non-breaking space | `10&nbsp;MB`, `SunCulture&nbsp;AMT`   | `10 MB` (may wrap)           |
| Loading text       | "Saving…"                             | "Saving..."                  |
| Number columns     | `font-variant-numeric: tabular-nums`  | Default numeric rendering    |
| Heading wrap       | `text-wrap: balance` or `text-pretty` | Default (widow words)        |

---

## Content handling

- Text containers must handle long content: `overflow: hidden` + `textOverflow: 'ellipsis'`,
  `line-clamp-*` (Tailwind), or `wordBreak: 'break-word'` for unbreakable strings.
- Flex children containing text need `minWidth: 0` — otherwise flexbox prevents
  truncation from working.
- Anticipate short, average, and very long user inputs in every layout.
  Test with a single character and a 200-character string.
- Handle empty states (see `references/ui-patterns.md`).

---

## Performance

- Virtualise lists exceeding 50 items (`react-window` or `@tanstack/virtual`).
- Avoid layout reads in render cycles: `getBoundingClientRect`, `offsetHeight`,
  `scrollTop` — read in `useEffect` or event handlers, not during rendering.
- Batch DOM reads and writes; use `requestAnimationFrame` for visual updates.
- Prefer uncontrolled inputs (`defaultValue`) where possible. Controlled inputs
  must be cheap to update on every keystroke.
- Add `<link rel="preconnect">` for CDN and asset origin domains.
- Critical fonts: `<link rel="preload" as="font" crossOrigin="anonymous">` +
  `font-display: swap`.
- Next.js `<Image>`: use `sizes` prop to serve appropriately sized images.

---

## Navigation and URL state

- URL must reflect UI state: active filters, active tab, current page number,
  open drawer / panel.
- Navigation links must use `<a>` / `<Link>` — not `onClick` handlers — to
  support Cmd/Ctrl+click, middle-click, and right-click → "Open in new tab".
- Deep-link all stateful UI. A URL copied from the browser should reproduce
  the same view.

**AMT — tab URL sync:**

```tsx
void router.replace({ query: { ...router.query, tab: key } }, undefined, {
  shallow: true,
});
```

**Mopesa — URL-synced state:**

```tsx
import { useSearchParams, useRouter } from "next/navigation";
// or: use nuqs for type-safe URL state
import { useQueryState } from "nuqs";
```

---

## Touch and interaction

- `touch-action: manipulation` on interactive elements to remove the 300ms
  tap delay on mobile.
- Set `-webkit-tap-highlight-color` intentionally (usually `transparent` for
  custom-styled elements).
- `overscroll-behavior: contain` inside modals and drawers to prevent scroll
  chaining.
- Disable text selection on draggable elements; mark them `inert` while drag
  is active.

---

## Safe areas and layout

- Full-bleed layouts need `env(safe-area-inset-*)` padding for notched devices.
- Prevent unwanted horizontal scrollbars: `overflow-x: hidden` on the page root.
- Prefer Flexbox / CSS Grid over JS measurement for layout.

---

## Dark mode and theming

**Mopesa** uses `next-themes` for dark/light mode:

```tsx
// app/layout.tsx
<ThemeProvider attribute="class" defaultTheme="system" enableSystem>
```

- Set `color-scheme: dark` on `<html>` for dark themes to get native dark
  scrollbars and form controls.
- Match `<meta name="theme-color">` to the current background.
- Explicit `background-color` and `color` on native `<select>` (it doesn't
  inherit CSS variables in all browsers).
- Use shadcn/ui CSS variable convention (`--background`, `--foreground`,
  `--primary`, etc.) — never hardcode dark-mode colours.

**AMT** does not support dark mode. Do not add dark-mode CSS to AMT without
explicit design approval.

---

## Locale and internationalisation

**AMT** uses `next-i18next`. Translation strings live in `public/locales/`.

- Use `Intl.DateTimeFormat` for locale-aware date/time rendering.
- Use `Intl.NumberFormat` for numbers and currency.
- AMT canonical date format: `DD MMM YYYY HH:mm` via `moment()`.
  Always guard: `moment(iso).isValid() ? moment(iso).format(...) : '—'`.
- Wrap brand names, code tokens, and identifiers with `translate="no"` to
  prevent machine translation from mangling them.
- Detect user language via `Accept-Language` header or `navigator.languages`.

---

## Hydration safety

- Controlled inputs need `onChange` (or use `defaultValue` for uncontrolled).
- Guard date/time rendering against hydration mismatch — `new Date()` returns
  different values on server vs. client. Use a `mounted` state or `useEffect`
  to gate time-sensitive renders.
- `suppressHydrationWarning` is a last resort. Document why it's needed inline
  when used.

---

## Anti-pattern checklist

Flag and fix all of the following:

| Anti-pattern                                                 | Problem                              | Fix                                 |
| ------------------------------------------------------------ | ------------------------------------ | ----------------------------------- |
| `user-scalable=no` / `maximum-scale=1`                       | Breaks a11y zoom                     | Remove from viewport meta           |
| `onPaste` with `preventDefault`                              | Blocks paste for password managers   | Remove                              |
| `transition: all`                                            | Unintended transitions + perf        | List specific properties            |
| `outline: none` / `outline: 0` without `:focus-visible` ring | Fails keyboard a11y WCAG AA          | Add visible focus ring              |
| `<div onClick>` / `<span onClick>`                           | Not keyboard accessible              | Use `<button>` or `<a>`             |
| Images without `width` + `height`                            | Layout shift (CLS)                   | Add dimensions                      |
| Lists > 50 items without virtualisation                      | Perf / render time                   | Virtualise                          |
| Form controls without labels                                 | Screen reader inaccessible           | Add `<label>` or `aria-label`       |
| Icon-only buttons without `aria-label`                       | Screen reader inaccessible           | Add `aria-label`                    |
| Hardcoded date format strings                                | Locale issues                        | Use `moment()` or `Intl`            |
| Unjustified `autoFocus`                                      | Disruptive, breaks mobile            | Desktop + single input only         |
| `[...map.values()]` / `[...set]`                             | TS `downlevelIteration` error in AMT | `Array.from(map.values())`          |
| `SweetAlert2` / `Swal.fire`                                  | Being phased out                     | `useConfirm()`                      |
| Empty state without next action                              | Unhelpful UX                         | Add context + actionable link       |
| Silent async failure                                         | User has no idea what happened       | `message.error(...)` in every catch |
