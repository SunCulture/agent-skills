# AGENTS.md

Guidance for AI coding agents working in this repository.

## Repository Overview

A monorepo of agent skills for SunCulture. Each skill is a self-contained
directory under `skills/` with a `SKILL.md` and optional `references/`,
`rules/`, `evals/`, or `scripts/`.

## Creating a New Skill

### Directory Structure

Skills are organised by category under `skills/`. Each category gets its own
subdirectory, and individual skills live inside it:

```
skills/
  {category}/             # e.g. typescript, python, go, web, devops
    {skill-name}/         # kebab-case skill name
      SKILL.md            # Required: frontmatter + instructions
      references/         # Optional: detailed reference docs
        {topic}.md
      rules/              # Optional: modular rule files
        {rule-name}.md
      evals/              # Optional: test cases
        evals.json
      scripts/            # Optional: executable scripts
        {script-name}.sh
```

### SKILL.md Frontmatter

```yaml
---
name: { skill-name } # kebab-case, max 64 chars, must match the skill directory name (not the language directory)
description: > # max 1024 chars, no angle brackets
  One or more sentences describing what the skill does and when to trigger it.
  Include example trigger phrases.
license: MIT
metadata:
  author: sunculture
  version: "1.0.0"
---
```

Allowed frontmatter keys: `name`, `description`, `license`, `allowed-tools`,
`metadata`, `compatibility`.

### Content Guidelines

- Keep `SKILL.md` under 500 lines — move deep reference material to `references/`
- Write specific, trigger-phrase-rich descriptions so the agent activates the
  skill at the right time
- Use progressive disclosure: `SKILL.md` gives the overview, `references/`
  files give the depth
- Include concrete code examples with before/after patterns where relevant

### Validation

```bash
python3 -m scripts.quick_validate /path/to/skills/{category}/{skill-name}
```

## Naming Conventions

- Category directory: `kebab-case` (e.g., `typescript`, `python`, `web`, `devops`)
- Skill directory: `kebab-case` (e.g., `functional-typescript`, `react-patterns`)
- `SKILL.md`: always uppercase, always this exact name
- Reference files: `kebab-case.md`
- Rule files: `{category}-{rule}.md` (e.g., `core-pure-functions.md`)
- Scripts: `kebab-case.sh`

## Commits

Follow conventional commits. Use the skill name (not the language) as the scope.
All commits to `main` trigger semantic-release:

- `feat(functional-typescript): ...` → minor release
- `fix(functional-typescript): ...` → patch release
- Breaking change footer → major release
