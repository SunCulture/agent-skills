# agent-skills

A collection of agent skills for AI coding assistants at SunCulture. Skills are
packaged instructions that extend agent capabilities with domain-specific
knowledge and patterns.

Skills follow the [Agent Skills](https://agentskills.io/) format and work with
Claude Code, Cursor, Copilot, and other AI coding agents.

## Available Skills

### functional-typescript

Helps write idiomatic functional TypeScript and JavaScript. Covers closures,
partial application, currying, pure functions, immutability, higher-order
functions, function composition, and the module pattern — grounded in *You
Don't Know JS Yet* foundations with TypeScript type system coverage.

**Use when:**
- Writing functional-style TypeScript or JavaScript
- Typing higher-order functions, curried functions, or generics
- Structuring code with the module pattern
- Avoiding mutation and side effects
- Understanding closures and scope

## Skill Structure

Each skill lives under `skills/` and follows this layout:

```
skills/
  {skill-name}/
    SKILL.md          # Required: metadata + instructions
    references/       # Optional: detailed reference docs
    scripts/          # Optional: executable helper scripts
    evals/            # Optional: test cases for the skill
    rules/            # Optional: modular rule files
```

## Installation

```bash
npx skills add https://github.com/sunculture/agent-skills --skill <skill>
```

Or manually copy a skill directory into your agent's skills folder.

## Contributing

1. Create a new directory under `skills/` (kebab-case)
2. Add a `SKILL.md` with frontmatter (`name`, `description`)
3. Follow conventional commits — releases are automated via semantic-release
4. Open a PR against `main`

## License

MIT
