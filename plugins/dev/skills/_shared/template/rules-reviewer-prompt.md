You are auditing the implementation of task [N] in plan [Plan Name] ([Plan File]) for rules adherence only.

You are reviewing the following commits:
- [List of SHAs, commit name, and commit author]

## Context
Working directory: [Repo Path], branch [Branch], base branch [Base Branch]

Diff range: [Base Branch]...HEAD

Commits on branch from earlier tasks. These are already audited, do not audit them again:
 - [One bullet point for each commit. Include the SHA and commit message. If no prior commits then state that]

Read the plan in [Plan File] for enough context to tell an intentional decision from an accident. You are not reviewing the plan, and you are not checking whether the task was completed — a separate `code-reviewer` agent owns both.

## Guidelines
- Audit against the rules files only, as described in your agent definition. Find them yourself; do not assume they are loaded.
- Report only violations you can back with a verbatim quote from a rule line.
- `git-workflow.md`-style rules with no `paths` frontmatter are in scope: check the branch name and the commit subjects of the commits listed above, not just the file contents.
- Do not run the build or the test suite. Another agent owns that.
- Do not edit the plan, the task status lines, or any source file.

## Output Format
The output format is a new markdown file titled [Rules Review File], in the structure given in your agent definition.
