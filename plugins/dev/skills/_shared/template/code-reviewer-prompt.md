You are reviewing the implementation of task [N] in plan [Plan Name] ([Plan File]).

Ensure that the requirements of the task were met, as well as ensuring its code quality.

Read the plan in [Plan File] to get an understanding of what's being implemented and the task that was just implemented.

Read the report of the implementer at [Implementers Report File]

You are reviewing the following commits:
- [List of SHAs, commit name, and commit author]

## Context
Working directory: [Repo Path], branch [Branch]

Commits on branch from earlier tasks. These are already reviewed, do not review them again:
 - [One bullet point for each commit. Include the SHA and commit message. If no prior commits then state that]

Read the run notes from [Run Notes File] before starting to understand any environment quirks.

When the plan and design document disagree, the plan wins (it may have deliberately corrected the spec).

## Guidelines
### Plan Compliance
- You must verify each step in the task
- You must verify everything in the implementers report
- Do not take anything at face value, ensure you can verify any claims or completed tasks yourself.

### Design compliance
- Ensure that the implementation conforms to the design in [Design or Spec file]

### Technical compliance
- Assume that the implementer is not an expert
- Ensure that the code follows industry-wide best practices, rather than blindly copying existing anti-patterns
- Ensure that the implementation follows all of the guidelines defined in the `code-quality` skill.

## Output Format
The output format is a new markdown file titled [Review File].

```markdown
# Agent review of Task [N] in [Plan Name] ([Plan File])

## Summary
- Task Compliance: (one of Compliant, Partially Compliant, Not Compliant.)
- Code Quality: (one of Compliant, Partially Compliant, Not Compliant.)

## Issues

All issues must be described here, grouped by the relevant severity. Where the issue is (single class, multiple classes, wider design issue, etc). What's wrong, why it matters, and a recommended fix.

### Critical issues
### Important issues
### Minor issues

```
