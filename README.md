# agent-skills

A collection of agent skills for AI coding assistants at SunCulture. Skills are
packaged instructions that extend agent capabilities with domain-specific
knowledge and patterns.

Skills follow the [Agent Skills](https://agentskills.io/) format and work with
Claude Code, Cursor, Copilot, and other AI coding agents.

## Available Skills

### typescript / functional-typescript

Helps write idiomatic functional TypeScript and JavaScript. Covers closures,
partial application, currying, pure functions, immutability, higher-order
functions, function composition, and the module pattern — grounded in _You
Don't Know JS Yet_ foundations with TypeScript type system coverage.

**Use when:**

- Writing functional-style TypeScript or JavaScript
- Typing higher-order functions, curried functions, or generics
- Structuring code with the module pattern
- Avoiding mutation and side effects
- Understanding closures and scope

### web / sunculture-frontend-design

Applies SunCulture brand guidelines, design tokens, and UX patterns when
building or reviewing web interfaces. Covers the teal/green/yellow colour
system, Circular Std typography, canonical gradient system, AMT design tokens
(`@/styles/theme`), Mopesa/TailwindCSS mapping, SunCulture voice and copy
rules, and UI patterns (forms, tables, detail pages, confirmation dialogs).

**Use when:**

- Styling a component or page on AMT or Mopesa frontend
- Reviewing code for brand alignment
- Writing UI copy for any SunCulture surface
- Building a new feature on a SunCulture product

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
