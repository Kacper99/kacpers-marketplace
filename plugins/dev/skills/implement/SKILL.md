---
name: implement
description: Use this skill when implementing a plan
user-invocable: true
disable-model-invocation: false
argument-hint: [plan directory]
arguments: plan_dir
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
Read the plan at $plan_dir — `context.md` plus every `task-NN.md` — along with the design document it was built from. The plan may already be in progress, so get a base understanding of where we're at by looking at the status of each task. Ensure that any in progress, or completed tasks do actually exist. These may be commits on the current branch, other branches, or already be merged into master. If an in progress or completed task cannot be found, stop and ask me about it.

Create the RUN-NOTES.md next to the plan if it does not exist. Use `template/RUN-NOTES.md` template to seed it.

Read the plan's dependency graph and compute the **independent chains**: sets of tasks that share no dependency. Record them in RUN-NOTES before starting, and note the top-level paths each chain touches so collision risk is visible.

Independent chains run **concurrently**, one engineer per chain, each dispatched with `isolation: "worktree"`. A worktree gives each chain its own working tree *and its own build directory*, which is what makes the concurrency safe. Serialise only within a chain, and never run two agents in the same worktree.

If the plan does not state its chains, derive them yourself and record what you derived.

## Step 2
Run the independent chains concurrently. Within a chain, work the tasks in order:
1. Dispatch a `engineer` subagent to implement the task in the plan. Use the `template/implementer-prompt.md` template to prompt the subagent.
2. As soon as the agent completes, set the status of the task to "Dev Complete"
3. Review the agents report from the task, if the agent has not raised any issues or concerns, set the status to "Ready for review"
4. Run the **gate build** yourself: one full build and test run, redirected to a file, reading the real exit code. This is the only full-suite run for this task. Pass the exit code and per-suite test counts to the reviewer so it does not repeat them.
5. Dispatch a `code-reviewer` agent to review the implementation. Use `template/code-reviewer-prompt.md` as the template to prompt the agent.
6. Once the code reviewer completes, review its comments and determine how to address them. Anything non-trivial, ask me for feedback.
7. If the reviewer comes back with any critical or important findings, dispatch a new `engineer` subagent (or re-use the one from the implementation step if they're still available — it still holds the context, which is materially cheaper than a fresh agent) to address the comments with the recommended solutions. Set the status of the task to "Addressing Review"
8. Once the Engineer completes addressing the review comments, verify that the build and tests still pass. If the failures are non trivial dispatch the agent again to resolve them. Set the status to completed once done.

**You own the Status lines.** Engineers and reviewers must not edit them; if one does, correct it.

**One build at a time per worktree.** Never start a build while an agent is building in the same worktree. Concurrent runs corrupt the test-results state, after which a suite fails within seconds carrying an I/O error that reads exactly like a broken test. Suspect this whenever a suite fails implausibly fast, confirm it by reproducing on a clean checkout with changes stashed, and fix it by deleting that suite's test-results directory — it is never a code change.

**Watchdog.** If an agent returns without committing, or reports waiting on something twice, take the task over yourself rather than resuming it a third time. Record in the task report that the agent did not complete it.

## Step 3
Once the plan is fully implemented, dispatch a workflow for an adversarial review. This pass is the most expensive thing in the run and the least productive per token, so keep it small unless the evidence says otherwise.

Always include, regardless of plan size:
- One code-quality finder over the entire branch diff, applying the code-quality skill. Give it the plan's global constraints and cross-task seams to check as part of its brief. This is the highest-yield auditor, because it is the only one that can see duplication and drift *across* tasks.
- A build-and-test verifier, the **only** agent permitted to run the build. Tell every other agent in the workflow explicitly not to invoke it, for the same corruption reason as Step 2. Expect this agent to find no defects: its job is confirmation, so do not count it as a finder.

Add a per-task plan-compliance finder ONLY for a task that needed a dispatched **Addressing Review** round. A task whose review came back clean, or whose minor findings you fixed directly, has already had its exact-compliance pass; do not re-audit it.

Run a separate constraints/seams auditor as its own agent only when the plan has a cross-cutting refactor touching more than half the tasks. Otherwise those checks belong in the code-quality finder's brief — run standalone it tends to re-find what the task-scoped and code-quality agents already have.

Verification of findings:
- Critical/important findings get 2 independent refuters, and **both** must refute to kill a finding.
- A **1-1 split**, or a finding raised independently by **2 or more auditors**, is escalated to me to adjudicate. Never silently dropped, and never auto-confirmed either: dissent and convergence are both signals that a human should look. Convergence in particular tracks "this is a real pattern in the code" rather than "this is worth fixing".
- Record minor findings without a verify pass.

# Reports
Reports should be placed under a report directory in the same directory the plan is in. The format is <plan-dir>/reports/task-N-{report,review}.md

Record a task's commit SHA only once that task is finally green, after any post-review amend. A SHA captured before a fix round will not exist by the end of the run.

# Selecting the subagents model
When spawning an `engineer` subagent you may only use Sonnet or Opus. Do not use Haiku or Fable. Select Sonnet or Opus based on the complexity of the task. Simple file moves or small changes can be handled by Sonnet.

When spawning a `code-reviewer` subagent. You may only use Opus.
