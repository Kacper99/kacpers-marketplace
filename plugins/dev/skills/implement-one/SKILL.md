---
name: implement-one
description: Use this skill when implementing a plan directly, without dispatching an engineer subagent per task
user-invocable: true
disable-model-invocation: false
argument-hint: [plan directory]
arguments: plan_dir
---

Execute a plan by implementing each task yourself, in the current agent, instead of dispatching an `engineer` subagent per task. This trades the isolation and concurrency of `implement` for lower overhead on plans that don't need it — small plans, or a single task you want done inline without spinning up a fresh agent.

Task review still goes to an independent `code-reviewer` agent, and the plan still gets the adversarial review pass at the end — both benefit from a perspective that didn't just write the code, and neither is expensive enough to justify collapsing into you.

# Roles

## Main Agent (you)
You implement every task directly, and you orchestrate + own the plan's Status lines, same as `implement` would.

## Code Reviewer
Use the `code-reviewer` agent for reviewing a task after you've completed its implementation. Same as `implement`.

# Steps

## Step 1
Read the plan at $plan_dir — `context.md` plus every `task-NN.md` — along with the design document it was built from. The plan may already be in progress, so get a base understanding of where we're at by looking at the status of each task. Ensure that any in progress, or completed tasks do actually exist. These may be commits on the current branch, other branches, or already be merged into master. If an in progress or completed task cannot be found, stop and ask me about it.

Create the RUN-NOTES.md next to the plan if it does not exist. Use `template/RUN-NOTES.md` to seed it.

Read the plan's dependency graph and compute the **independent chains**, same as `implement` does. There's no concurrency or worktree isolation here — everything runs serially, in this working directory — so the chains aren't a dispatch unit. They're there to tell you what's ready to pick up next: once you finish a task, stay on its chain if it has a ready follow-on, otherwise hop to the next chain with a ready task. Record the chains in RUN-NOTES before starting.

If the plan does not state its chains, derive them yourself and record what you derived.

## Step 2
Work tasks one at a time, in the current agent, picking each one per the chain ordering from Step 1:

1. **Implement the task yourself.** Read the task and design doc, read RUN-NOTES for environment quirks, and ask me now about anything unclear in the requirements or approach before you start — there's no subagent to stop and ask instead of you. Then implement exactly what the task specifies, in the plan's repo/branch. Set the status to "Dev Complete" once the implementation itself is done.
2. **Verify as you go, narrowly.** Run the narrowest thing that answers your question — a single test class, one suite — while working. Reserve the full build for one gate run immediately before committing. Never background a build and poll it; run it in the foreground, redirect to a file, and read the real exit code:

       check_log=$(mktemp); <build command> > "$check_log" 2>&1; echo "EXIT=$?"

   Use `mktemp` rather than a fixed name like `/tmp/check.log` — a fixed, predictable path in shared `/tmp` can collide with another run. Do not consider it green until you've seen that exit code — a cached build can report up to date without running a single test.
3. **Self review**, same bar `implement`'s engineer holds itself to: everything in the spec implemented, project compiling, affected tests passing (checking a test still exists is not the same as checking it still pins the same behaviour — a test can keep its name and cover less), formatting/build checks run. Fix anything you find immediately; this is mechanical, don't dispatch a reviewer for it. Commit, then set the status to "Ready For Review".
4. **Write the task report** to `<plan-dir>/reports/task-N-report.md`: what was implemented, what tests were added and their status, what Agent Skills/Rules were used, self-review findings (if any), deviations from the plan.
5. **Run the gate build** (if step 2's last narrow run wasn't already the full suite): one full build and test run, redirected to a file, reading the real exit code. Note the exit code and per-suite counts for the reviewer so it doesn't repeat them.
6. **Dispatch a `code-reviewer` agent** to review the implementation, using `template/code-reviewer-prompt.md`.
7. Once the reviewer completes, review its comments yourself and determine how to address them. Anything non-trivial, ask me for feedback.
8. If the reviewer raises critical or important findings, address them yourself, directly — there's no separate engineer to hand this back to. Set the status to "Addressing Review" while you do.
9. Once addressed, verify the build and tests still pass. Set the status to "Completed" once done. Record the task's commit SHA now, after any post-review amend — a SHA captured before a fix round won't exist by the end of the run.
10. **Continue or checkpoint.** If you still have plenty of context budget left and there's a ready next task (per the chains from Step 1), move straight on to it without waiting to be asked. If context is getting tight, or nothing is ready (blocked on a decision, or the plan's exhausted), stop here and report where things stand — the plan and RUN-NOTES are enough for a fresh run of `implement-one` to pick back up.

**You own the Status lines.** The code reviewer must not edit them; if it does, correct it.

**If you find yourself stuck on the same thing twice** — waiting on something, or a fix that doesn't hold — stop and ask me rather than attempting it a third time.

## Step 3
Once the plan is fully implemented, dispatch a workflow for an adversarial review, identical to `implement`'s Step 3:

Always include, regardless of plan size:
- One code-quality finder over the entire branch diff, applying the code-quality skill. Give it the plan's global constraints and cross-task seams to check as part of its brief. This is the highest-yield auditor, because it is the only one that can see duplication and drift *across* tasks.
- A build-and-test verifier, the **only** agent permitted to run the build. Tell every other agent in the workflow explicitly not to invoke it, for the same corruption reason as Step 2. Expect this agent to find no defects: its job is confirmation, so do not count it as a finder.

Add a per-task plan-compliance finder ONLY for a task that needed an "Addressing Review" round. A task whose review came back clean, or whose minor findings you fixed directly, has already had its exact-compliance pass; do not re-audit it.

Run a separate constraints/seams auditor as its own agent only when the plan has a cross-cutting refactor touching more than half the tasks. Otherwise those checks belong in the code-quality finder's brief — run standalone it tends to re-find what the task-scoped and code-quality agents already have.

Verification of findings:
- Critical/important findings get 2 independent refuters, and **both** must refute to kill a finding.
- A **1-1 split**, or a finding raised independently by **2 or more auditors**, is escalated to me to adjudicate. Never silently dropped, and never auto-confirmed either: dissent and convergence are both signals that a human should look. Convergence in particular tracks "this is a real pattern in the code" rather than "this is worth fixing".
- Record minor findings without a verify pass.

# Reports
Reports should be placed under a report directory in the same directory the plan is in. The format is <plan-dir>/reports/task-N-{report,review}.md

Record a task's commit SHA only once that task is finally green, after any post-review amend. A SHA captured before a fix round will not exist by the end of the run.

# Selecting the code-reviewer's model
When spawning a `code-reviewer` subagent you may only use Opus. There's no engineer subagent to pick a model for — you implement in whatever model you're already running as.
