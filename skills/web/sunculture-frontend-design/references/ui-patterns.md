# SunCulture UI Patterns

Canonical UX patterns for SunCulture web interfaces. Framework-agnostic —
apply the principles in any stack.

---

## Mobile-first layout considerations

Every pattern below must work on a 320px–767px viewport first.

- Stack form fields vertically on mobile; use multi-column only from `tablet` up.
- Dialogs/modals: bottom sheet or full-screen on mobile; centred modal from `tablet` up.
- Tables: card list on mobile; data table from `tablet` up (or use horizontal scroll
  with `overflow-x: auto` as a minimum).
- Navigation: bottom-aligned primary actions on mobile; sidebar/top-nav from `tablet` up.

---

## Forms

### Layout rules

- `layout="vertical"` — label above field at all viewports.
- Explicit, specific validation message on every required field.
  Never rely on a generic default from a library.
- Full-width inputs on mobile. Constrain width from `tablet` up where appropriate.
- Fields with non-obvious meaning: add helper text beneath the input.
- Remote-data selects: show a loading indicator in the placeholder while data fetches.
- Number inputs: enforce `min`, `max`, and `step` constraints. Convert to number before submit.

### Submit flow: validate → confirm → mutate

This flow applies to every create, update, and destructive action:

```
1. Validate all fields client-side
   → surface errors inline, focus the first failing field
   → abort if any field is invalid

2. Normalise inputs
   → trim whitespace from strings
   → coerce number fields to the correct type
   → run any optional pre-checks (duplicate name, eligibility)
   → surface warnings before proceeding

3. Confirm
   → show a confirmation dialog (see section below)
   → abort if user cancels

4. Mutate
   → submit to API
   → show success feedback on completion
   → show descriptive error feedback on failure (surface server message if available)
   → never swallow errors silently
```

### Edit-mode initialisation

Pre-fill all fields before the form becomes visible — not lazily on open.
For create mode: reset to empty state and optionally seed contextual defaults
(e.g. inherit a value from a parent entity).

### Dialog / modal guidance

- Compact form (3–5 fields): `max-width: 520px`; full-screen on mobile.
- Multi-section form: `max-width: 880px`; full-screen on mobile.
- Always dismiss and reset state when the dialog closes.
- On mobile: prefer a bottom sheet that slides up from the screen edge.
- Primary action button: right-aligned (or full-width on mobile), loading state while submitting.
- Secondary (cancel) button: left-aligned, always visible.

---

## Confirmation dialogs

Every create / update / destructive / side-effect action requires an explicit
confirmation step. This is not optional.

### Copy rules

| Property      | Rule                                                                   |
| ------------- | ---------------------------------------------------------------------- |
| Title         | A clear yes/no question — "Save changes to this pay plan?"             |
| Description   | Names the entity AND the most consequential values being changed       |
| Confirm label | Action verb — "Create", "Save changes", "Deactivate". Never "OK"       |
| Cancel label  | "Back to form" when triggered from a form; "Cancel" for action buttons |
| Destructive   | Visually distinct — danger colour on the confirm button                |

### When confirmation is required

- Creating a record.
- Updating a record.
- Any destructive action (deactivate, delete, bulk-change status).
- Any action with downstream side effects (sync jobs, notifications, financial transactions).

### Mobile presentation

On narrow viewports, confirmation dialogs should be:

- Full-screen overlay, **or**
- Bottom sheet anchored to the screen edge

Avoid centred floating dialogs on mobile — they compete with the soft keyboard.

---

## Data tables (list pages)

### Shell structure

```
┌─ Card / container ──────────────────────────────┐
│  ┌─ Filter row ─────────────────────────────┐   │
│  │  [label] [input]  [label] [select]  ...  │   │
│  └──────────────────────────────────────────┘   │
│  ┌─ Table ──────────────────────────────────┐   │
│  │  [view] [ID] [Name] [Status] [Date] ...  │   │
│  └──────────────────────────────────────────┘   │
│  ┌─ Pagination ─────────────────────────────┐   │
└─────────────────────────────────────────────────┘
```

### Column rules

| Column            | Rules                                                    |
| ----------------- | -------------------------------------------------------- |
| First — view link | Icon-only, `aria-label="View"`, navigates to detail page |
| ID / reference    | Centre-aligned                                           |
| Name / title      | Clickable link to detail page                            |
| Status            | Badge or tag (success / neutral / danger colour)         |
| Boolean           | `'Yes'` / `'No'` — never `true`/`false`                  |
| Missing value     | `'—'` (em dash) — never blank, never `null`              |
| Date              | `DD MMM YYYY HH:mm` — validate before formatting         |

### Pagination and filtering

- Server-side pagination — never load all rows client-side.
- Any filter change resets to page 1.
- Export always uses the active filter set.
- Filter state persists in URL query params (deep-linkable).
- Label each filter control with visible text above it.

### Mobile table approach

On mobile (< 768px), prefer one of:

1. **Card list** — render each row as a stacked card with key fields visible.
2. **Horizontal scroll** — `overflow-x: auto` with a minimum column width
   so the table doesn't collapse illegibly.
3. **Progressive disclosure** — show key columns only; expand row for details.

Never render a complex multi-column table without any mobile adaptation.

---

## Detail pages with tabs

### Page structure

```
┌─ Hero section ──────────────────────────┐
│  [eyebrow] [Title]   [Action buttons]   │
└─────────────────────────────────────────┘
┌─ Tab bar ───────────────────────────────┐
│  [Overview] [Config] [History] ...      │
└─────────────────────────────────────────┘
┌─ Tab panel ─────────────────────────────┐
│  [Loading placeholder]                  │
│    → [Empty state with next action]     │
│    → [Content]                          │
└─────────────────────────────────────────┘
```

### Tab key rules

- Define a typed allowlist of valid tab keys.
- Validate the URL param against the allowlist before applying it.
- Sync the active tab to the URL — the page must be deep-linkable.
- Conditional tabs (feature-gated): reset to the default tab when the active
  key no longer exists for the current entity.

### Tab panel rules

- Each tab panel is a separate, dedicated component — not inlined on the page.
- Render order inside a panel: loading state → empty state → content.
- Empty state: name what is missing AND offer a next action (don't just say "No data").
- On mobile: tabs may collapse into a select/dropdown when there are > 4.

### Mobile detail pages

- Hero section stacks vertically on mobile.
- Action buttons move to a sticky bottom bar or a "…" overflow menu on mobile.
- Tab bar scrolls horizontally on mobile (no wrapping).

---

## Empty states

Every empty state must do two things:

1. Name specifically what is missing.
2. Offer a clear, actionable next step.

```
✓ "No lease configuration yet. Set up lease config →"
✗ "No data."
✗ Empty component with no text.
```

If the user lacks permission to take the next action, explain why and what
they can do instead (e.g. "Contact your admin to set this up").

---

## Loading and error states

**Loading:**

- Show a skeleton or spinner covering the content area.
- Include a short "Loading…" label for screen readers.

**Success:**

- Always show a success notification after a mutation completes.
- Name the entity that was created/updated in the message.

**Error:**

- Always show an error notification on failure.
- Surface the server's error message if it is human-readable.
- Never swallow errors silently.

---

## Permissions

- **Hide** (don't disable) actions the user cannot perform.
- The detail page itself (read view) stays visible to read-only users.
- Check permissions server-side — client-side hiding is UX only, not security.
- On mobile: hidden actions should not leave empty space — collapse or reflow layout.

---

## API conventions

- One module per domain: a service file for request functions and a types file
  for record/payload shapes.
- Strip `undefined` values from query params before sending.
- Handle Decimal / JSON numeric types from the API before displaying in UI.
- Accept both `snake_case` and `camelCase` in response types defensively.
- Wire format: match what the API uses (usually `snake_case`); normalise
  to `camelCase` internally after parsing.
