---
allowed-tools: Bash(git status:*), Bash(git diff:*), Bash(git log:*), Bash(git rev-parse:*), Bash(git branch:*), Bash(git blame:*), Bash(git show:*), Bash(git rev-list:*), Bash(git symbolic-ref:*)
description: Code review the current git branch
disable-model-invocation: false
---

Provide a code review for the current git branch compared to the base branch.

The user may optionally provide a base branch name as an argument (e.g., `/code-review-branch origin/develop`). If not provided, auto-detect the base branch by trying `origin/main`, then `origin/master`, then using `git symbolic-ref refs/remotes/origin/HEAD`.

To do this, follow these steps precisely:

1. Determine the base branch to compare against. If the user provided a base branch argument, use that. Otherwise, auto-detect using the following logic:
   - Try `git rev-parse --verify origin/main` - if it exists, use `origin/main`
   - Otherwise try `git rev-parse --verify origin/master` - if it exists, use `origin/master`
   - Otherwise use `git symbolic-ref refs/remotes/origin/HEAD | sed 's@^refs/remotes/@@'`
   - If all fail, error and ask the user to specify a base branch

2. Check for safety issues:
   - Get the current branch name with `git rev-parse --abbrev-ref HEAD`
   - If the current branch is `main` or `master`, error and inform the user that they cannot review the main branch
   - Check if there are uncommitted changes with `git diff-index --quiet HEAD --` - if there are, warn the user that only committed changes will be reviewed

3. Use a Haiku agent to check if the current branch (a) has no changes compared to the base branch, (b) is already merged into the base branch, or (c) has only trivial changes that don't need review (e.g., fewer than 5 lines changed). If so, do not proceed. Use these git commands:
   - Check if branch has changes: `git diff --quiet <base>...HEAD` (if exit code is 0, no changes)
   - Check if merged: `git branch --merged <base> | grep -q "$(git rev-parse --abbrev-ref HEAD)"`
   - Get change statistics: `git diff --shortstat <base>...HEAD`

4. Use another Haiku agent to give you a list of file paths to (but not the contents of) any relevant CLAUDE.md files from the codebase: the root CLAUDE.md file (if one exists), as well as any CLAUDE.md files in the directories whose files the branch modified. Use:
   - `git diff --name-only <base>...HEAD` to get changed files
   - Check for CLAUDE.md files in those directories

5. Use a Haiku agent to analyze the git diff and commit history, and ask the agent to return a summary of the changes. Use:
   - `git rev-parse --abbrev-ref HEAD` to get branch name
   - `git log <base>..HEAD --pretty=format:"%s"` to get commit messages
   - `git diff --stat <base>...HEAD` to get file change summary
   - `git diff --shortstat <base>...HEAD` to get lines added/removed

6. Then, launch 4 parallel Sonnet agents to independently code review the change. The agents should do the following, then return a list of issues and the reason each issue was flagged (eg. CLAUDE.md adherence, bug, historical git context, etc.):
   a. Agent #1: Audit the changes to make sure they comply with the CLAUDE.md files found in step 4. Note that CLAUDE.md is guidance for Claude as it writes code, so not all instructions will be applicable during code review. Use `git diff <base>...HEAD` to get the changes.
   b. Agent #2: Read the file changes using `git diff <base>...HEAD`, then do a shallow scan for obvious bugs. Avoid reading extra context beyond the changes, focusing just on the changes themselves. Focus on large bugs, and avoid small issues and nitpicks. Ignore likely false positives.
   c. Agent #3: Read the git blame and history of the code modified, to identify any bugs in light of that historical context. Use `git diff --name-only <base>...HEAD` to get changed files, then `git log -p -- <file>` and `git blame <file>` for each changed file.
   d. Agent #4: Read code comments in the modified files, and make sure the changes comply with any guidance in the comments. Use `git diff <base>...HEAD` to get changes, then read the full file contents to see all comments.

7. For each issue found in step 6, launch a parallel Haiku agent that takes the branch diff, issue description, and list of CLAUDE.md files (from step 4), and returns a score to indicate the agent's level of confidence for whether the issue is real or false positive. To do that, the agent should score each issue on a scale from 0-100, indicating its level of confidence. For issues that were flagged due to CLAUDE.md instructions, the agent should double check that the CLAUDE.md actually calls out that issue specifically. The scale is (give this rubric to the agent verbatim):
   a. 0: Not confident at all. This is a false positive that doesn't stand up to light scrutiny, or is a pre-existing issue.
   b. 25: Somewhat confident. This might be a real issue, but may also be a false positive. The agent wasn't able to verify that it's a real issue. If the issue is stylistic, it is one that was not explicitly called out in the relevant CLAUDE.md.
   c. 50: Moderately confident. The agent was able to verify this is a real issue, but it might be a nitpick or not happen very often in practice. Relative to the rest of the changes, it's not very important.
   d. 75: Highly confident. The agent double checked the issue, and verified that it is very likely it is a real issue that will be hit in practice. The existing approach in the branch is insufficient. The issue is very important and will directly impact the code's functionality, or it is an issue that is directly mentioned in the relevant CLAUDE.md.
   e. 100: Absolutely certain. The agent double checked the issue, and confirmed that it is definitely a real issue, that will happen frequently in practice. The evidence directly confirms this.

8. Filter out any issues with a score less than 80. If there are no issues that meet this criteria, do not proceed.

9. Use a Haiku agent to repeat the eligibility check from step 3, to make sure that the branch still has changes and hasn't been merged during the review.

10. Finally, output the review results to the terminal in formatted markdown. When writing your output, keep in mind to:
   a. Keep your output brief
   b. Avoid emojis
   c. Include absolute file paths with line numbers (e.g., `/absolute/path/to/file.ts:42-48`)
   d. Include code snippets for context
   e. Reference the commit SHAs for reproducibility

Examples of false positives, for steps 6 and 7:

- Pre-existing issues
- Something that looks like a bug but is not actually a bug
- Pedantic nitpicks that a senior engineer wouldn't call out
- Issues that a linter, typechecker, or compiler would catch (eg. missing or incorrect imports, type errors, broken tests, formatting issues, pedantic style issues like newlines). No need to run these build steps yourself -- it is safe to assume that they will be run separately as part of CI.
- General code quality issues (eg. lack of test coverage, general security issues, poor documentation), unless explicitly required in CLAUDE.md
- Issues that are called out in CLAUDE.md, but explicitly silenced in the code (eg. due to a lint ignore comment)
- Changes in functionality that are likely intentional or are directly related to the broader change
- Real issues, but on lines that were not modified in the branch

Notes:

- Do not check build signal or attempt to build or typecheck the app. These will run separately, and are not relevant to your code review.
- Make a todo list first
- You must cite and reference each bug (eg. if referring to a CLAUDE.md, you must reference it with the absolute file path)
- For your final output, follow the following format precisely (assuming for this example that you found 2 issues):

---

### Code Review

Branch: `feature/new-feature`
Base: `main` (abc123de)
Head: def456ab
Files: 8 changed (+127/-43)

Found 2 issues:

#### 1. Missing error handling for async operation

**Confidence:** 85/100
**Source:** Bug detection
**File:** `/Users/kacpermartela/projects/kacpers-marketplace/src/handler.ts:42-48`

```typescript
async function processData(input: string) {
  const result = await fetchData(input);
  return result.data;
}
```

**Issue:** No try-catch block for async operation. If fetchData fails, unhandled promise rejection will occur.

#### 2. Violates CLAUDE.md naming convention

**Confidence:** 90/100
**Source:** CLAUDE.md compliance (`/Users/kacpermartela/projects/kacpers-marketplace/CLAUDE.md`)
**File:** `/Users/kacpermartela/projects/kacpers-marketplace/src/utils.ts:15-20`

CLAUDE.md states: "All utility functions must use camelCase naming"

```typescript
export function Process_Data(input: string): string {
  return input.trim();
}
```

**Issue:** Function name uses snake_case instead of camelCase.

---

- Or, if you found no issues:

---

### Code Review

Branch: `feature/new-feature`
Base: `main` (abc123de)
Head: def456ab
Files: 8 changed (+127/-43)

No issues found. Checked for bugs and CLAUDE.md compliance.

---

- Additional formatting notes:
  - Use absolute file paths for all file references
  - Include line number ranges for context (e.g., `:42-48`)
  - Include code snippets using markdown code blocks with language syntax highlighting
  - Reference the base and head commit SHAs (short form is fine)
  - List the file change statistics (files changed, insertions, deletions)
