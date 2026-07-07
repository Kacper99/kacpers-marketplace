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
6. Dispatch a new `engineer` subagent to address the comments with the recommended solutions. Set the status of the task to "Addressing Review"
7. Once the Engineer completes addressing the review comments, set the status to completed

## Step 3
Once the plan has been fully implemented, dispatch a workflow to perform an adversarial review of the implementation. Ensure that agents check the plan has been followed exactly, and that the implementation conforms to the standards set out in the `code-quality` plugin.
