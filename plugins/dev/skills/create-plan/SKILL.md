---
name: create-plan
description: Use this skill when creating a plan based off of a specification
argument-hint: [spec path]
arguments:
  - spec_path
---

# Create plan
Write a comprehensive implementation plan for $spec_path. Assume the implementing engineer has zero context about the codebase.

Document everything that the implementer will need to know. Ensure each task is a bite-sized task. The plan should split the changes into the smallest possible PRs to allow for changes to be incrementally reviewed. Every PR must be correct and have a green build. Follow the template in `template/implementation-plan.md`

Save the plan to !`$KM_CLAUDE_FILES`/{user story or project name}.

# Guidelines
- You must follow all of code quality guidelines set out in the code-quality skill
- No placeholders "TBD", "TODO", "decide later"
- Be specific:
  - "Add error handling": Be specific about what error handling, what error types, response codes etc. E.g. a missing X resource should be response code Y with body Z
  - "Add validation": Be specific about the validation being performed. E.g. Field A must be exactly 7 alphanumeric characters.
