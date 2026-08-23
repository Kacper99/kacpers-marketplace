You are implementing Task [N] in plan [Plan Name]

Read the plan from [Plan File]. This contains the full context of the plan. You may also refer to the design document, [Design or Spec file], if needed.

## Context
Working directory: [Repo Path], branch [Branch]

Commits on branch from earlier tasks:
 - [One bullet point for each commit. Include the SHA and commit message. If no prior commits then state that]

Read the run notes from [Run Notes File] before starting to understand any environment quirks. If you add an entry, it must cite the command or file that proves it — an unevidenced claim there will be read as fact by every later task. If you find an existing entry is wrong, correct it in place and say so rather than adding a contradicting one.

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

Do not edit the plan's **Status** lines. The orchestrator owns them.

## Verification budget

While working, run the **narrowest thing that answers your question**: a single test class for one behaviour, one suite for a suite-level change. Reserve the full check-everything build for **one run immediately before you commit**. A single class often runs in seconds where the full build takes minutes, and most questions during implementation are single-class questions.

Never run two builds at once, and never start one in the background and poll for it. Run it in the foreground, redirect the output to a file, and read the real exit code:

    check_log=$(mktemp); <build command> > "$check_log" 2>&1; echo "EXIT=$?"

Use `mktemp` rather than a fixed name like `/tmp/check.log` — other chains build concurrently in their own worktrees, and a fixed name in shared `/tmp` collides across them. Do not report a build as green unless you have seen its exit code. Grepping console output for a success string is not the same thing, and a cached build can report everything up to date without executing a single test.

## Self Review
Review your work and ensure:
- Fully implemented everything in the spec
- If writing code:
  - the project is compiling
  - any affected tests are passing
  - All formatting and build check steps have been run


Checking that every *test* exists is not the same as checking that every *behaviour* is pinned. A test can keep almost the same name and cover less than it used to.

If you find any issues during self review, fix them immediately. This is a mechanical check, do not spawn a separate reviewer agent for this.

## Report format
Write a full report to [Report File]:
- What was implemented
- What tests were added and their status (passing/not passing)
- What Agent Skills and Agent Rules were used
- Self review findings (if any)
- Any deviations from the original plan.
