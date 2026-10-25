---
name: release
description: >
  Cherry-pick a set of tickets from stage (or another source branch/commit)
  onto a release branch, verify with lint/test/build, open a release PR, then
  tag and publish a GitHub release. Use when the user says "make a release",
  "cut a release", "cherry-pick these tickets onto the release branch", or names
  a version to release (e.g. "release v3.2.11").
license: MIT
allowed-tools:
  - Bash(git *)
  - Bash(gh *)
  - Bash(npm *)
  - Read
  - Glob
  - AskUserQuestion
metadata:
  author: sunculture
  version: "1.0.0"
---

# /release — cherry-pick a release, verify, PR, tag, publish

Arguments passed: `$ARGUMENTS` — expected to contain some combination of:

- a release/target branch name (or "current branch" if already checked out)
- a list of ticket IDs (e.g. `RETAIL-706 RETAIL-687 RETAIL-710`)
- optionally one or more explicit commit SHAs to include regardless of ticket
- optionally a version number for the tag/release (e.g. `v3.2.11`)

If any of these are missing or ambiguous, ask rather than guess — this skill
makes real commits and opens real PRs/releases.

## 1. Resolve tickets to commits

`git log --grep` only matches commit message bodies. Most feature branches
name the ticket in the **branch name** or **PR title** — the individual
commits inside often say nothing about it. A three-pass search is required:

**Pass A — commit message grep:**

```
git log --branches --remotes --oneline --grep="<TICKET>" -i
```

**Pass B — branch name search:**

```
git branch -a | grep -i "<TICKET>"
```

For every matching branch, collect the branch's own commits, measured against
the **source** branch (e.g. `origin/stage`), not the release target. A branch
cut from the source can look hundreds of commits "ahead" of a target that has
diverged from it, which floods the list with unrelated work; Step 2 decides
what is already on the target. Skip `release/*` branches and the branch you are
building:

```
git log origin/<source-branch>..<branch> --oneline   # commits not yet on the source
```

If a branch is already merged, find its merge commit:

```
git log --branches --remotes --oneline --merges | xargs -I{} git log -1 --format="%H %s" {} | grep -i "<TICKET>"
```

Then expand it: `git log <merge>^1..<merge>^2 --oneline`

**Pass C — PR title and body search (requires gh):**

```
gh pr list --state all --search "<TICKET> in:title,body" --json number,title,headRefName,mergeCommit
```

For any PR not already covered by Pass A or B, expand its merge commit the
same way. Also look up the PRs for every branch found in Pass B, because a PR
can omit the ticket from both its title and body:

```
gh pr list --state all --head <branch> --json number,title,mergeCommit
```

**Union all three passes** before proceeding. If a ticket's commits are
found in **any** pass — A, B, or C — they must be included. A result from
Pass B or C carries exactly the same weight as one from Pass A; do not
treat a commit as optional or uncertain just because it wasn't surfaced by
the grep. The only valid reason to exclude a commit is `git merge-base
--is-ancestor` confirming it is already on the release target.

Rules that apply to every pass:

- A ticket may have **multiple PRs and multiple branches** (rework, follow-up
  fixes, test coverage, docs). Treat every PR and every branch found as its
  own full unit — expand each one completely, not just the first.
- **For each PR (merge commit) found**: expand to all its content commits with
  `git log <merge>^1..<merge>^2 --oneline`. Never cherry-pick the merge
  commit itself.
- **For each branch found**: collect every commit on that branch not yet on
  the source branch with `git log origin/<source>..<branch> --oneline`. If the
  branch was already merged, find its merge commit and expand that instead.
- Do not stop after finding one PR or one branch for a ticket. Run all passes
  to completion and union every commit found before moving on.
- If a ticket has **zero matches** in all three passes, do not invent a
  commit or silently skip it. Tell the user it wasn't found and ask how to
  proceed (they may supply the right PR/branch/SHA, or say to skip it).
- If the user supplied an explicit commit SHA directly, include it as-is
  alongside whatever the search finds.
- **Deduplicate by SHA** once the passes are unioned. The same commit often
  turns up under several tickets, passes, PRs and branches. Keep one entry per
  SHA, record every ticket it satisfies, and never cherry-pick a SHA twice.
- **Keep a ledger** per ticket: every resolved SHA, which pass found it, and
  later its outcome (picked / already on target / equivalent patch / dropped
  with a reason). The completion audit in section 3 depends on it.
- **Ignore non-work and promotions.** `git stash` entries (subjects starting
  `index on` or `WIP on`) are not commits to pick. Promotion and squash commits
  or PRs such as `Promote Develop to Stage`, `Merge develop to stage` and
  `chore(stage): promote …` bundle unrelated work: do not pick or expand them,
  and do not expand any PR whose base is the source branch. The ticket's own
  commits are picked individually, so record the promotion in the ledger as
  dropped (promotion).
- **Shared PRs.** A PR whose body names several tickets (an epic or batch PR)
  is expanded once and its commits are credited to every ticket it names.
  Count each SHA once in totals.

## 2. Check for already-applied work

Before cherry-picking anything, for every resolved commit run:

```
git merge-base --is-ancestor <commit> <release-branch> && echo YES || echo NO
```

Do this for **every** commit, not only suspected ones. A commit that is not an
ancestor can still be on the target under a different hash (picked or rebased
earlier). Compare patch IDs:

```
# patch IDs already on the target (run once)
git log <release-branch> --no-merges -p --format=commit%n%H | git patch-id --stable
# patch ID of the candidate
git show <commit> | git patch-id --stable
```

A matching patch ID is a no-op; note the target-side SHA in the ledger. Patch
IDs differ when the earlier copy needed a conflict resolution, so for those
fall back to the check below.

If a patch ID doesn't match but you still suspect the work is already present under a different hash
(same commit message/author/timestamp appearing in `git log <release-branch>`
history is a strong signal — this happens when a branch was previously
promoted piecemeal), **don't trust the signal alone**. Confirm with a real
diff against the actual source:

```
git diff <release-branch> origin/<source-branch> -- <paths the commit touches>
```

An empty diff proves it's a no-op. Report which tickets are no-ops and skip
cherry-picking them (don't create empty commits unless the user asks for an
explicit record of having picked them).

## 3. Cherry-pick in chronological order

Sort **all** resolved, non-ancestor commits by commit date across every
ticket (not per-ticket in isolation — interleave them), then:

```
git cherry-pick -x <sha>
```

one at a time, in that order. `-x` records `(cherry picked from commit <sha>)` in
each new commit so the audit below can trace every pick back to its source.

### Completion audit (required, before Verify)

Picking is not finished until every SHA in the ledger is accounted for. After
the last pick, walk the ledger and, for each ticket and each resolved SHA,
confirm exactly one outcome:

- **Picked**: `git log <release-branch> --grep="cherry picked from commit <sha>" -n1`
  finds it.
- **Equivalent patch already on target**: `git cherry <release-branch> <sha> <sha>^`
  prints `-` (`+` means no patch-equivalent commit is on the branch; a
  conflict-resolved copy also shows `+`, so check the `-x` trail first).
- **Dropped**: only for a reason you can state (merge commit, user said skip,
  unrelated commit inside a shared PR) and have reported to the user.

Any SHA with no outcome is a missed pick: go back and resolve it. Do not move
on to Verify, and do not report the release as complete, until every ticket
has a full ledger. End the run with a table per ticket of SHAs found, picked,
already present and dropped, and carry it into the PR body.

## 4. Conflict resolution policy

- **Default**: on a plain textual conflict, keep the incoming (cherry-picked)
  side — it represents the newer/intended change.
- **Stop and ask the user** (`AskUserQuestion`, with concrete evidence —
  line ranges, dates, `git show <commit> --stat`) when a conflict is bigger
  than a textual clash:
  - a modify/delete conflict touching a file unrelated to the ticket's
    actual scope
  - a conflict block spanning hundreds of lines because the source commit
    predates heavy refactors on this branch
  - the incoming change depends on fields/helpers/model columns that were
    never carried onto this branch (a missing prerequisite) — in that case
    find the commits on the source branch that introduced them (`git log
    <source> -S'<symbol>' -- <path>`, or the file's history on the source
    branch) that are not yet on the target, and **replay those prerequisite
    commits first, oldest first**, instead of hand-writing a patch or
    dropping lines. Don't do it silently: tell the user which extra commits
    were pulled in and why. Ask before replaying only when the chain is large
    (more than about 10 commits) or would bring back a feature that was
    deliberately reverted on the target; then offer replay, a smaller
    targeted patch, or dropping the dependent lines as out of scope.
- After resolving a conflict, stage the files and continue with hooks
  skipped (this skill's default — see Guardrails):
  `HUSKY=0 git cherry-pick --continue` (note: `cherry-pick --continue` does
  **not** accept `--no-verify` directly; `HUSKY=0` is the working bypass for
  husky-based hooks — check `.husky/` first in case this repo uses a
  different mechanism).
- Beware `git checkout --ours`/`--theirs` silently deleting or reintroducing
  content in files _not_ part of the conflict markers you're resolving —
  after resolving a same-feature conflict, re-diff the affected area against
  the source branch (e.g. `git diff HEAD origin/<source-branch> -- <path>`)
  to confirm it still matches before moving to the next commit.

## 5. Verify

Run, in order: `npm run lint`, `npm run build`, `npm test`.

For build and test failures, **don't assume they're caused by this release**.
This repo's branches can carry pre-existing baseline failures unrelated to
any given release:

- Capture a baseline: detached-checkout the release branch's pre-cherry-pick
  tip (`git checkout <original-tip-sha>`), run `npx tsc -p tsconfig.build.json --noEmit`
  and count errors, then return to the branch tip (`git checkout <branch>`).
- Compare that count against the post-cherry-pick count. Only newly
  introduced errors are this release's problem — report pre-existing ones
  as known baseline noise, not a regression.
- For test failures, check the failure reason before flagging it — a
  missing native binding (e.g. `sqlite3`) or other local-environment issue
  is not the same as a real regression from the cherry-picked code.

## 6. Push and open the PR

- `git push -u origin <branch> --no-verify` (hooks skipped by default).
- Look for `.github/PULL_REQUEST_TEMPLATE.md` (or lowercase variant) and
  fill in its exact sections if present.
- If the branch has no upstream and the right base isn't obvious (e.g. a
  release-candidate branch could target `develop` or `main`), ask the user
  rather than guessing — get this wrong and the diff/PR is meaningless.
- **Write the PR description in plain, non-technical language** by default
  for this skill's release PRs: one bullet per user-facing change, phrased
  in terms of what changed for someone using the product, not ticket IDs or
  implementation details. This is the release-specific default — it differs
  from `/commit-and-create-pr`'s more technical PR body.
- Note any no-op tickets, not-found tickets and replayed prerequisite commits
  in the PR body so reviewers know they were considered, and include the
  ledger summary table from the completion audit.
- Return the PR URL to the user.

## 7. Tag + GitHub release

- Look up the most recent release to copy its exact format before creating
  a new one:
  ```
  gh release list --limit 1
  gh release view <that-tag>
  ```
  Match its title pattern, tag pattern, and body structure exactly (as of
  this skill being written: title `Release Candidate v<version>`, tag
  `v<version>`, body with a `## What's new` section of plain-language
  bullets and a `## Notes` line confirming build/lint checks ran clean).
- Ask the user which commit the tag should point to — the release branch's
  tip, or the base branch's tip (this depends on whether the PR from step 6
  has merged yet; don't assume).
- Create and push the tag:
  ```
  git tag <version> <target-commit>
  git push origin <version> --no-verify
  ```
- Publish the release:
  ```
  gh release create <version> --title "Release Candidate <version>" --notes "<body>"
  ```
- Return the release URL to the user.

## Guardrails

- Never add a `Co-Authored-By:` trailer (for any author, human or tool) to
  commits or to PR/release bodies.
- Never force-push.
- This skill skips hooks by default (`--no-verify` / `HUSKY=0`) for all of
  its own git operations, since release branches routinely carry pre-existing
  baseline failures unrelated to the release itself. This is a local
  convenience only — never suggest bypassing hooks or checks in the actual
  CI pipeline.
- Never invent a ticket's commit when it can't be found — ask.
- Never resolve a large or ambiguous conflict silently — ask, with evidence.
- Never assume a build/test failure is caused by this release without
  checking against a baseline first.
- If `gh` isn't authenticated or there's no GitHub remote, say so and stop.
