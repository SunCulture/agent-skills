---
name: deslop-and-review
description: >
  Clean up AI-generated code slop and run a rigorous agent code review on the
  current branch or pull request. Use when the user asks to deslop code, clean
  up AI-generated changes, review agent-written code, prepare a PR for review,
  or run deslop and code review together.
license: MIT
metadata:
  author: sunculture
  version: "1.0.0"
---

# Deslop and Review

Run two distinct quality passes over a branch: first remove obvious
AI-generated slop, then review the remaining change for correctness,
maintainability, security, performance, and test coverage.

This skill follows the public `deslop` pattern from
[skills.sh](https://skills.sh/brianlovin/claude-config/deslop): compare against
the base branch, remove AI-generated bloat while preserving intended behaviour,
and keep the final cleanup summary short.

Full workflow details: [review workflow](references/review-workflow.md).

## When to Apply

Use this skill when:

- Cleaning up AI-generated code before a PR
- Reviewing code written by an agent or coding assistant
- Preparing a branch for human review
- Running "deslop and review" as a single workflow
- Auditing a PR for both slop and substantive review findings

## Core Principle

Keep cleanup and review separate.

The deslop pass should make minimal behaviour-preserving edits. The review pass
should identify bugs, regressions, missing tests, and maintainability issues. Do
not bury correctness findings inside style cleanup.

## Workflow

1. Gather the branch diff, changed files, and staged/unstaged state.
2. Run a behaviour-preserving deslop pass for AI-generated bloat.
3. Review the remaining diff for correctness, security, tests, maintainability,
   performance, and documentation impact.
4. Use parallel subagents for larger diffs when the host agent supports them.
5. Fix only what the user asked to fix, then run focused verification.

Read [review workflow](references/review-workflow.md) before applying fixes or
when the diff spans multiple files.

## References

- Detailed local workflow:
  [references/review-workflow.md](references/review-workflow.md)
- Public deslop skill:
  [skills.sh/brianlovin/claude-config/deslop](https://skills.sh/brianlovin/claude-config/deslop)
- Public code review examples:
  [skills.sh code-review search](https://www.skills.sh/?q=code-review)
