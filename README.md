# agent-skills

A collection of agent skills for AI coding assistants at SunCulture. Skills are
packaged instructions that extend agent capabilities with domain-specific
knowledge and patterns.

Skills follow the [Agent Skills](https://agentskills.io/) format and work with
Claude Code, Cursor, Copilot, and other AI coding agents.

This repo also ships a roster of specialist **[agents](agents/)** — subagents
named for the speciality they handle (architecture, security, SRE, code review,
and more). See [`agents/README.md`](agents/README.md) for the full roster.

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

### `devops/`

#### `deslop-and-review` · [view](https://github.com/SunCulture/agent-skills/tree/main/skills/devops/deslop-and-review)

Clean up AI-generated code slop and run a rigorous agent code review on the
current branch or pull request. Combines the public deslop workflow from
[skills.sh](https://www.skills.sh/?q=deslop) with structured review for
correctness, security, test coverage, maintainability, and performance.

**Use when** cleaning up AI-generated changes before a PR, reviewing
agent-written code, preparing a branch for human review, or running deslop and
code review together.

```bash
npx skills add https://github.com/SunCulture/agent-skills --skill deslop-and-review
```

### `engineering/`

#### `fullstack-engineering` · [view](https://github.com/SunCulture/agent-skills/tree/main/skills/engineering/fullstack-engineering)

Principal-level fullstack engineering patterns across stacks — Golang (Bun ORM,
BunRouter, adapter pattern), Python (hexagonal, FastAPI, SQLAlchemy 2.0),
C#/.NET (clean architecture, EF Core), TypeScript/Node.js, Next.js, Nuxt.js,
React, Vue, React Native, Kotlin, Swift, plus queues, databases, and infra.
Enforces SOLID/SRP, UUIDv7 primary keys, structured logging, OpenTelemetry
tracing, health/readiness probes, graceful shutdown, multi-stage non-root
Dockerfiles, real test coverage, and Makefiles in every output.

**Use when** writing, reviewing, debugging, refactoring, or scaffolding any
application, service, API, worker, frontend, mobile app, or infra config.

```bash
npx skills add https://github.com/SunCulture/agent-skills --skill fullstack-engineering
```

## Agents

Specialist subagents live under [`agents/`](agents/), one `.md` file per
speciality. They follow the Claude Code subagent format (frontmatter + operating
instructions) and are auto-discovered when copied into an agent directory. The
full roster, lifecycle grouping, and the codename → speciality mapping are
documented in [`agents/README.md`](agents/README.md).

```bash
cp agents/<speciality>.md ~/.claude/agents/
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
