---
name: sunculture-frontend-design
description: >
  Apply SunCulture brand guidelines, design tokens, UX patterns, mobile-first
  layout, accessibility standards, and web interface copy rules when building
  or reviewing web interfaces. Use when styling a component, checking UI for
  brand alignment, writing SunCulture web copy, designing forms, confirmation
  dialogs, data tables, detail pages, or building any SunCulture web interface
  regardless of framework.
license: MIT
metadata:
  author: sunculture
  version: "1.2.0"
---

# SunCulture Frontend Design

Source of truth for brand identity, design tokens, UI patterns, and web
interface standards across SunCulture web products.

This skill is framework-agnostic. Apply it to any web stack.

## When to Apply

Use this skill when:

- Building or reviewing any SunCulture web interface
- Styling components, layouts, states, or responsive behaviours
- Writing UI copy, empty states, errors, labels, or confirmation text
- Checking accessibility, focus states, animation, or performance
- Choosing colours, typography, spacing, radius, gradients, or imagery
- Designing forms, confirmation dialogs, data tables, or detail pages

## Core Rules

1. Build mobile-first. Write base styles for mobile and progressively enhance
   with `min-width` media queries.
2. Use SunCulture design tokens. Do not hardcode raw hex colours, raw spacing,
   raw font sizes, or one-off radius values in product UI.
3. Use Circular Std with Arial fallback. Do not use Inter, Roboto, Space
   Grotesk, or `system-ui` as the primary brand font.
4. Keep UI copy grounded, benefit-led, human, and in UK spelling.
5. Validate form input before confirmation, then mutate only after the user
   confirms the action.
6. Confirm every create, update, destructive, or side-effect action.
7. Hide actions users cannot perform. Do not rely on disabled controls for
   permission gating.
8. Use semantic HTML, visible labels, explicit focus states, and accessible
   names for icon-only controls.

## Required References

Read the reference file that matches the task before making detailed decisions:

- Brand foundation, colours, gradients, typography, logo, iconography, and
  photography: [references/brand-identity.md](references/brand-identity.md)
- Token values and implementation guidance for colour, spacing, typography,
  radius, sizes, and layout helpers:
  [references/design-tokens.md](references/design-tokens.md)
- Framework-agnostic UI patterns for forms, confirmation dialogs, data tables,
  detail pages, empty states, permissions, and API conventions:
  [references/ui-patterns.md](references/ui-patterns.md)
- Web standards for mobile-first layout, accessibility, semantic HTML,
  animation, content handling, performance, locale handling, hydration, dark
  mode, and anti-patterns:
  [references/web-standards.md](references/web-standards.md)

## Quick Checklist

Before finalising a web UI change, check:

- Mobile layout works from 320px wide and touch targets are at least 44px.
- Components use design token variables rather than raw values.
- Copy uses UK spelling, second person, active voice, and action verbs.
- Empty states explain what is missing and offer a next step.
- Forms show visible labels and explicit validation messages.
- Confirmation dialogs name the entity and the consequential change.
- Tables use server-side pagination and a mobile alternative when needed.
- Detail page tabs validate URL params and keep the URL in sync.
- Async success and failure states are visible to the user.
- Motion respects `prefers-reduced-motion` and avoids `transition: all`.

## Anti-Patterns

Avoid:

- Desktop-first layouts
- Hover-only controls
- Raw hex colours or raw pixel values in product styles
- Missing labels, missing `aria-label` values, or `outline: none`
- Confirmation buttons labelled "OK"
- Empty states that only say "No data."
- Loading every table row client-side
- Client-side permission checks without server-side enforcement
