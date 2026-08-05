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

Save the plan to !`echo "$KM_CLAUDE_FILES"`/{user story or project name}/plan, **as a directory of files rather than one document**:

    context.md      shared: goal, constraints, codebase orientation, dependency
                    graph, definition of done. Every agent reads this.
    task-01.md      one file per task. An agent reads context.md plus its own
    task-02.md      task file, and nothing else.
    ...

A single-file plan grows past what one read can hold, so every agent pays a paging and re-orientation tax on work that is not its own, and mid-flight corrections become risky to make.

# Guidelines
- You must follow all of code quality guidelines set out in the code-quality skill
- No placeholders "TBD", "TODO", "decide later"
- Be specific:
  - "Add error handling": Be specific about what error handling, what error types, response codes etc. E.g. a missing X resource should be response code Y with body Z
  - "Add validation": Be specific about the validation being performed. E.g. Field A must be exactly 7 alphanumeric characters.
- **Specify what a test must establish, not the literal assertion code** — unless you have actually run that code against the codebase. Prescribing an exact assertion you have not executed converts a planning error into a review round trip, and it happens easily: an assertion can be unreachable because the real code takes a path you did not model, or vacuously true because the collection it inspects is empty in the arranged scenario.
  - Good: "assert the run is priced at the reviewing pass's model — assert on the model, not on token totals, because a totals assertion still passes when the run is mispriced."
  - Bad: "assert `containsExactly(A, B, C)`" when you have not checked that exactly A, B and C occur.
- The same applies to **prose you tell an implementer to put in a comment or Javadoc**: verify the claim against the code before prescribing it. A confidently-worded false comment is worse than no comment, because it reads as licence to change the code it misdescribes.
