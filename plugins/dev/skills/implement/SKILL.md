---
name: implement
description: Use this skill when implementing a plan
user-invocable: true
disable-model-invocation: false
argument-hint: [plan file]
arguments: plan_filename
---

Execute a plan as an orchestrator dispatching subagents per task.

# Roles

## Main Agent (you)
You are responsible for orchestrating subagents to implement the plan, and updating the plan as the subagents perform their tasks.

## Engineer
Use the `engineer` agent for any implementation tasks. Each agent is responsible for implementing a single task.

## Code Reviewer
Use the `code-reviewer` agent for reviewing a task after an Engineer has completed its implementation.

# Steps

## Step 1
Read the design and plan files, the plan may already be in progress so get a base understanding of where we're at by looking at the status of each task. Ensure that any in progress, or completed tasks do actually exist. These may be commits on the current branch, other branches, or already be merged into master. If an in progress or completed task cannot be found, stop and ask me about it.

Create the RUN-NOTES.md next to the plan if it does not exist. Use `template/RUN-NOTES.md` template to seed it.

Determine whether the plan implementation would benefit from a workflow. Signals which suggest a workflow would be beneficial:
- Several independent tasks
- Major refactorings across several files

Signals which would not suggest a workflow:
- Small number of tasks

## Step 2
For each task in the plan:
1. Dispatch a `engineer` subagent to implement the task in the plan. Use the `template/implementer-prompt.md` template to prompt the subagent.
2. As soon as the agent completes, set the status of the task to "Dev Complete"
3. Review the agents report from the task, if the agent has not raised any issues or concerns, set the status to "Ready for review"
4. Dispatch a `code-reviewer` agent to review the implementation. Use `template/code-reviewer-prompt.md` as the template to prompt the agent.
5. Once the code reviewer completes, review its comments and determine how to address them. Anything non-trivial, ask me for feedback.
6. If the reviewer comes back with any critical or important findings, dispatch a new `engineer` subagent (or re-use the one from the implementation step if they're still available) to address the comments with the recommended solutions. Set the status of the task to "Addressing Review"
7. Once the Engineer completes addressing the review comments, verify that the build and tests still pass. If the failures are non trivial dispatch the agent again to resolve them. Set the status to completed once done.

## Step 3
Once the plan is fully implemented, dispatch a workflow for an adversarial review. Size it to risk:

Always include, regardless of plan size:
- A constraints/integration auditor checking the plan's global constraints and cross-task seams (no per-task review saw the whole).
- A build-and-test verifier re-running the full affected test set and any integration tests, checking real exit codes. It must re-run anything a per-task reviewer reported relying on a report for rather than executing.
- One code-quality finder over the entire branch diff, applying the code-quality plugin.

Add per-task plan-compliance finders ONLY for tasks that were not straightforwardly clean: a review verdict other than Compliant, an Addressing Review cycle, or reviewer-noted verification gaps. Tasks that went through review clean have already had an exact-compliance pass; do not re-audit them.

Verification of findings: verify critical/important findings adversarially (2 independent refuters, majority kills); record minor findings without a verify pass.

If any task ended Partially/Not Compliant, or the plan had major cross-cutting refactors, run the full pool (per-task finders for every task) instead.

# Reports
Reports should be placed under a report directory in the same directory the plan is in. The format is <plan-dir>/reports/task-N-{report,review}.md

# Selecting the subagents model
When spawning an `engineer` subagent you may only use Sonnet or Opus. Do not use Haiku or Fable. Select Sonnet or Opus based on the complexity of the task. Simple file moves or small changes can be handled by Sonnet.

When spawning a `code-review` subagent. You may only use Opus.
