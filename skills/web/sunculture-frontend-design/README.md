# sunculture-frontend-design

An agent skill for building and reviewing web interfaces that are on-brand,
accessible, and consistent across SunCulture products.

## What it does

When installed, this skill guides your AI coding assistant to apply SunCulture's
brand system — colours, typography, gradient palette, and design tokens —
alongside canonical UI patterns and web interface standards used across the
AMT and Mopesa frontends.

## Contents

```
skills/web/sunculture-frontend-design/
├── SKILL.md                  ← brand, tokens, patterns, standards (quick ref)
└── references/
    ├── brand-identity.md     ← brand promise, colour palette, gradients, typography,
    │                            logo rules, iconography, photography direction
    ├── design-tokens.md      ← AMT @/styles/theme token system + Mopesa/TailwindCSS
    │                            mapping (full tables, presets, CSS vars)
    ├── ui-patterns.md        ← forms, modals, confirmation dialogs, data tables,
    │                            detail pages with tabs, permissions, API conventions
    └── web-standards.md      ← accessibility, focus states, animation, performance,
                                 i18n, dark mode, hydration safety
```

## Installation

```bash
npx skills add https://github.com/sunculture/agent-skills --skill sunculture-frontend-design
```

Or copy `SKILL.md` and `references/` into your project's
`.agents/skills/sunculture-frontend-design/` directory, then add the skill to
your agent configuration.

## Covers

- SunCulture brand colours (teal `#07B282`, yellow `#F3D500`, green `#62C628`)
  and three canonical gradient sweeps
- Circular Std typography with Arial fallback
- Logo usage rules (min size, exclusion zone, don'ts)
- AMT design tokens (`@/styles/theme` — `semanticColors`, `space`, `fontSize`,
  `radii`, `sizes`, presets)
- Mopesa / TailwindCSS + shadcn/ui brand mapping
- Validate → confirm → mutate form submission pattern
- `useConfirm()` confirmation dialog requirements
- Data table canonical shape (`products.tsx` pattern with `useTablePagination`)
- Detail pages with typed tab allowlists and URL sync
- Permission-gated actions (`useFeaturePermissions`, `PermissionGate`)
- Voice and copy rules (UK spelling, benefit-led, "you" perspective)
- Accessibility, focus states, animation, and performance standards

## License

MIT
