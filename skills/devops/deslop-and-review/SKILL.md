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

### 1. Gather context

Inspect the branch before changing anything:

```bash
git status --short
git diff --stat origin/main...HEAD
git diff origin/main...HEAD
git diff --name-only origin/main...HEAD
```

If the repository uses a different base branch, compare against that branch
instead. Also inspect unstaged changes when the user asks to review local work.

### 2. Deslop pass

Check the diff against the base branch and remove AI-generated slop introduced
by the current branch.

Focus on:

- Extra comments that a human would not add or that do not match local style
- Defensive checks, fallbacks, or try/catch blocks that are abnormal for trusted
  or already-validated code paths
- Casts to `any` or broad `unknown` handling used only to bypass type issues
- Deep nesting that can be simplified with early returns or clearer structure
- Repeated helper wrappers, pass-through abstractions, or "just in case" code
- Naming, formatting, or phrasing that is inconsistent with nearby code

Guardrails:

- Preserve intended behaviour unless fixing a clear bug.
- Preserve intentional human-authored code and existing file conventions.
- Prefer focused edits over broad rewrites.
- Keep the cleanup summary to 1-3 sentences.

### 3. Agent code review pass

After the deslop pass, review the remaining diff as a reviewer.

Prioritize findings in this order:

1. Correctness bugs, regressions, and broken edge cases
2. Security issues, secrets, unsafe input handling, or authorization gaps
3. Test coverage gaps for changed behaviour
4. Maintainability regressions, spaghetti branching, or misplaced ownership
5. Performance risks such as N+1 queries, avoidable serial work, or large
   unnecessary recomputation
6. Documentation gaps for public APIs or operational behaviour
7. Low-value nits only when there are no higher-priority findings

For each finding, include:

- Severity: `critical`, `warning`, or `note`
- The affected file or symbol
- Why it matters
- A concrete suggested fix

If no substantive issues are found, say so clearly and note any residual test
gap or unverified assumption.

### 4. Use subagents when available

For larger diffs, run independent passes in parallel when the host agent
supports subagents:

- Deslop subagent: inspect the diff for AI slop and propose minimal cleanup.
- Code review subagent: inspect correctness, security, tests, performance, and
  maintainability.
- Strict maintainability subagent: when available, use a deep code-quality
  review for abstraction quality, file growth, and spaghetti branching.

The parent agent should merge the results, deduplicate findings, and fix or
report issues in priority order.

For small diffs, a single agent may run the passes sequentially.

### 5. Fix policy

When the user asks to apply fixes:

- Fix critical bugs, security issues, and broken tests before style cleanup.
- Apply deslop edits only where the cleanup is clearly behaviour-preserving.
- Avoid unrelated refactors.
- Re-run the most relevant tests, type checks, or linters after substantive
  edits.
- Report any checks that could not be run.

When the user asks for report-only review, do not edit files.

## Output Format

Use this structure:

```markdown
## Findings

- `critical` - Finding title in `path/to/file`
  Why it matters. Suggested fix.

- `warning` - Finding title in `path/to/file`
  Why it matters. Suggested fix.

## Deslop Summary

1-3 sentences summarising cleanup performed, or "No deslop edits made."

## Verification

Tests or checks run, with outcomes. If not run, explain why.
```

## References

- Public deslop skill:
  [skills.sh/brianlovin/claude-config/deslop](https://skills.sh/brianlovin/claude-config/deslop)
- Public code review examples:
  [skills.sh code-review search](https://www.skills.sh/?q=code-review)
