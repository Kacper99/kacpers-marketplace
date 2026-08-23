---
name: brainstorm
description: Use this for non-trivial creative or design work, especially when requirements, tradeoffs, or behavior are unclear.
---

# Brainstorming Ideas
Turn ideas into fully formed designs or specifications.

Use this for:
- New features
- Significant behavior changes
- Product/design decisions
- Ambiguous implementation requests
- Work with multiple plausible approaches

Do not force the full process for:
- Small, obvious edits
- Mechanical refactors
- Simple bug fixes
- Tasks where the user explicitly asks for a quick answer

# Process
1. Understand the request and project context. 
    - Identify purpose, constraints, stakeholders, affected areas, and success criteria.
    - Separate known facts from assumptions.
2. Interview me relentlessly. 
    - This should be continued until all ambiguity is removed. 
    - This step is expected to be several rounds of questions and discussions. This step is only short if the request is simple. 
3. Propose sensible approaches.
    - Prefer at least two approaches for meaningful decisions.
    - Explain tradeoffs, risks, complexity, and recommendation.
4. Present a design document based on the selected approach.
5. Write the document to !`echo "$KM_CLAUDE_FILES"`/{user story or project name}/spec.md
6. Spawn a separate agent to do an adversarial review of the specification. Ensure the agent checks for any contradictions, ambiguity, deviation from the original request, or incomplete steps.
7. Ask the user for feedback on the specification, and address any comments. Do not blindly accept each review comment, ensure you validate it before agreeing. You may push back on any comments.

# Guidelines
- Designs/Specifications should be high-level. We are not looking for specific code implementations. Pseudocode, or high-level examples are enough.
