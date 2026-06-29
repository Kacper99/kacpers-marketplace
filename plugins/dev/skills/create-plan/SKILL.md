---
name: create-plan
description: Use this skill when creating a plan based off of a specification
argument-hint: [spec path]
arguments:
  - spec_path
---

# Create plan
Write a comprehensive implementation plan for $spec_path. Assume the implementing engineer has zero context about the codebase.

Document everything that the implementer will need to know. Ensure each step is a bite-sized task. The plan should split the changes into the smallest possible PRs to allow for changes to be incrementally reviewed. Every PR must be correct and have a green build.

Save the plan to !`$KM_CLAUDE_FILES`/{user story or project name}.
