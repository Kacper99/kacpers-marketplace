---
name: rules-check
description: Audit a branch, or the uncommitted working tree, against the rules files in ~/.claude/rules and the repo's .claude/rules. Use when asked to check rules adherence, to check whether a change follows the rules, or to check a branch before pushing.
user-invocable: true
disable-model-invocation: false
argument-hint: [base branch]
arguments:
  - base_branch
---

Audit a change for rules adherence on demand, outside a plan run.

This skill is a launcher. The audit procedure — finding the rules files, matching their `paths` globs, the bar for a finding, the report shape — lives in `../../agents/rules-reviewer.md`. That file is the single source of truth and is written to stand on its own in any harness. Do not repeat it here, and do not reimplement it.

# Dispatch, do not audit inline

Dispatch the `rules-reviewer` agent even when you could do the work yourself. You may be the agent that wrote the code under review, and an author auditing their own work in a context full of competing instructions is the exact failure this exists to catch. The separate context window with one narrow brief is the whole mechanism.

# Steps

## Step 1 — Resolve the base branch
Use `$base_branch` if it was given. Otherwise auto-detect, in order:
- `git rev-parse --verify origin/main`
- `git rev-parse --verify origin/master`
- `git symbolic-ref refs/remotes/origin/HEAD | sed 's@^refs/remotes/@@'`

If all three fail, stop and ask for a base branch.

## Step 2 — Resolve what to audit
Get the current branch with `git rev-parse --abbrev-ref HEAD`, and check for uncommitted changes with `git diff-index --quiet HEAD --`.

- **On a feature branch, working tree clean.** Audit `<base>...HEAD`. This is the normal case.
- **On a feature branch, working tree dirty.** Tell me what is uncommitted, then audit the branch *and* the uncommitted changes — a pre-push check that ignores unstaged work checks the wrong thing. Brief the agent with both the range and the output of `git diff HEAD`.
- **On `main` or `master` with a dirty working tree.** There is no branch to audit. Audit the uncommitted changes alone, and say that is what you did.
- **On `main` or `master` with a clean working tree.** There is nothing to audit. Stop and say so.

If `git diff --quiet <base>...HEAD` reports no changes and the tree is clean, stop — do not dispatch an agent to review an empty diff.

## Step 3 — Dispatch
Brief the `rules-reviewer` with the working directory, the current branch, the base branch, the diff range, and the commit list from `git log --format='%H %s' <base>..HEAD`. The commit list is not optional: rules files with no `paths` frontmatter govern branch names and commit subjects, and the agent cannot check those from a diff.

Name no output file. The agent returns its report directly when none is named, which is what you want here — an on-demand check should not litter the repo with review files. Write one only if I ask for it.

## Step 4 — Report
Relay the agent's report to the terminal, in full, including its "Rules checked" table. The table is how I know which rules were actually in scope; a summary that drops it is worse than useless, because a rule silently judged out of scope looks identical to a rule that passed.

Do not fix anything. This skill reports; it does not edit. If I want the findings addressed, I will ask, and that is a separate pass.
