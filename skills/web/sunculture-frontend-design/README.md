# sunculture-frontend-design

An agent skill for building and reviewing web interfaces that are on-brand,
accessible, mobile-first, and consistent across all SunCulture products.

## What it does

When installed, this skill guides your AI coding assistant to apply SunCulture's
brand system — colours, typography, gradient palette, and design tokens — alongside
a mobile-first approach, canonical UI patterns, and web interface standards.
Framework-agnostic: applies to any web stack.

## Contents

```
skills/web/sunculture-frontend-design/
├── SKILL.md                  ← brand, tokens, mobile-first, patterns, standards
└── references/
    ├── brand-identity.md     ← brand promise, colour palette, gradients, typography,
    │                            logo rules, iconography, photography direction
    ├── design-tokens.md      ← full token values (colours, spacing, typography,
    │                            radius, sizes) as CSS custom properties + gradients
    ├── ui-patterns.md        ← forms, confirmation dialogs, data tables, detail pages
    │                            with tabs, permissions, mobile adaptations
    └── web-standards.md      ← mobile-first, accessibility, focus states, animation,
                                 performance, i18n, dark mode, anti-pattern checklist
```

## Installation

```bash
npx skills add https://github.com/SunCulture/agent-skills --skill sunculture-frontend-design
```

Or copy `SKILL.md` and `references/` into your project's
`.agents/skills/sunculture-frontend-design/` directory.

## Covers

- Mobile-first layout approach (breakpoints, touch targets, thumb-friendly patterns)
- SunCulture brand colours (`#07B282` teal, `#F3D500` yellow, `#62C628` green)
  and three canonical gradient sweeps
- Circular Std typography with Arial fallback
- Logo usage rules (min size, exclusion zone, don'ts)
- Full design token system as CSS custom properties
- Validate → confirm → mutate form submission pattern
- Confirmation dialog requirements (title, description, label copy)
- Data table canonical shape (columns, pagination, mobile card list)
- Detail pages with typed tab allowlists and URL sync
- Permission-gated actions (hide, don't disable)
- Voice and copy rules (UK spelling, benefit-led, "you" perspective)
- Accessibility, focus states, animation (`prefers-reduced-motion`), performance

## License

MIT
