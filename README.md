# agent-skills

A collection of agent skills for AI coding assistants at SunCulture. Skills are
packaged instructions that extend agent capabilities with domain-specific
knowledge and patterns.

Skills follow the [Agent Skills](https://agentskills.io/) format and work with
Claude Code, Cursor, Copilot, and other AI coding agents.

## Available Skills

### `typescript/`

#### `functional-typescript` · [view](https://github.com/SunCulture/agent-skills/tree/main/skills/typescript/functional-typescript)

Functional programming patterns for TypeScript and JavaScript — pure functions,
immutability, closures, currying, partial application, higher-order functions,
function composition, and the module pattern.

**Use when** writing functional-style TypeScript, typing HOFs or curried
functions, structuring code with the module pattern, or avoiding mutation and
side effects.

```bash
npx skills add https://github.com/SunCulture/agent-skills --skill functional-typescript
```

### `web/`

#### `sunculture-frontend-design` · [view](https://github.com/SunCulture/agent-skills/tree/main/skills/web/sunculture-frontend-design)

SunCulture brand guidelines, design tokens, mobile-first approach, and UX
patterns for web interfaces. Covers the teal/green/yellow colour system,
Circular Std typography, full token system as CSS custom properties, and
canonical patterns for forms, data tables, detail pages, and confirmation dialogs.

**Use when** building or reviewing any SunCulture web interface, writing UI
copy, or checking a component for brand alignment.

```bash
npx skills add https://github.com/SunCulture/agent-skills --skill sunculture-frontend-design
```

## Skill Structure

Skills are organised by category. Each skill lives under `skills/{category}/`
and follows this layout:

```
skills/
  {category}/               # e.g. typescript, python, web, devops
    {skill-name}/
      SKILL.md              # Required: metadata + instructions
      references/           # Optional: detailed reference docs
      scripts/              # Optional: executable helper scripts
      evals/                # Optional: test cases for the skill
      rules/                # Optional: modular rule files
```

## Installation

```bash
npx skills add https://github.com/sunculture/agent-skills --skill <skill>
```

Or manually copy a skill directory into your agent's skills folder.

## Contributing

1. Create a new directory under `skills/{category}/{skill-name}/` (both kebab-case)
2. Add a `SKILL.md` with frontmatter (`name`, `description`)
3. Follow conventional commits — releases are automated via semantic-release
4. Open a PR against `main`

## License

MIT
