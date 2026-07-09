You are implementing Task [N] in plan [Plan Name]

Read the plan from [Plan File]. This contains the full context of the plan. You may also refer to the design document, [Design or Spec file], if needed.

## Context
Working directory: [Repo Path], branch [Branch]

Commits on branch from earlier tasks:
 - [One bullet point for each commit. Include the SHA and commit message. If no prior commits then state that]

Read the run notes from [Run Notes File] before starting to understand any environment quirks.

When the plan and design document disagree, the plan wins (it may have deliberately corrected the spec).

## Before you begin
If you have any questions about:
- The wider plan
- The requirements
- The approach
- Anything else which is unclear

Then stop and ask them now.

## Your job

Once you're clear on the requirements:
- Implement exactly what the task specifies
- Once you've done the implementation, perform a self review
- Write a report on what you've done.
- Once the report is written, report back with a short summary (not the full content of the report)

## Self Review
Review you work and ensure:
- Fully implemented everything in the spec
- If writing code:
  - the project is compiling
  - any affected tests are passing
  - All formatting and build check steps have been run

If you find any issues during self review, fix them immediately. This is a mechanical check, do not spawn a seperate reviewer agent for this.

## Report format
Write a full report to [Report File]:
- What was implemented
- What tests were added and their status (passing/not passing)
- What Agent Skills and Agent Rules were used
- Self review findings (if any)
- Any deviations from the original plan.
