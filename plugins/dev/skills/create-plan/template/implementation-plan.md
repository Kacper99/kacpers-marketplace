# [Feature Name] implementation plan

**Goal: ** [Short description of what this plan is building]

## Contstrains
[Generic constraints that apply to this problem. Trade offs, limitations, requirements. Should be sourced from the spec.]

## Dependency Graph
[Create a mermaid diagram to show the dependencies between tasks]

## Tasks

### Task [N]: [Task name]
[Tasks should ideally be scoped to small single change. The change should be a single PR and be passing on its own]

**Status: ** [Ready For Dev, In Dev, Dev Complete, Ready for Review, In Review, Review Complete, Addressing Review, Completed]

**Dependencies: ** [what tasks this does this one depend on. What specific functionality/classes/interfaces does it rely on]

**Produces: ** [What this task achieves and which tasks will rely on this functionality]

#### Step [1-A]: [Write failing tests]
[Which tests to create, What type of tests, what to cover, what to assert. Only include this is if there are tests to write.]

#### Step [A+1]: Run tests to verify their failing 
Run ONLY the new tests we added and ensure they are failing

[Can be omitted if previous steps do not add tests.]

#### Step [A+2 - B]: Write implementations
[Steps describing what functionality/classes/interfaces to write and implement, what commands need to be run, or what general work needs to be carried out. If its a larger coding task, a minimal implementation is preferred here to satisfy the requirements.]

#### Step [B+1]: Run tests to verify
Run the tests we added and ensure they're passing. Iterate until tests are passing

[Can be omitted if previous steps do not add tests.]

#### Step [B+2]: Refactor
Refactor the implementation to follow best practices

[Can be omitted if non-coding task, or if a smaller change]

#### Step [B+3]: Commit
Commit the implementation and tests.
