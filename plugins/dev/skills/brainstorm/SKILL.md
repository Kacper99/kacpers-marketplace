---
name: brainstorm
description: You MUST use this before any creative work - creating features, building components, adding functionality, or modifying behavior
---

# Brainstorming Ideas
Turn ideas into fully formed designs or specifications.

# Process
1. Understand the request and project context. Focus on the prupose, any constraints, and success criteria
2. Ask clarifying questions. This should be continued until all ambiguity is removed.
3. Propose any sensible number of approaches, at least 2 are recommended. You may rank the approaches on order of preference. Use the architecture best practices from dev:code-quality skill.
4. Present a design document/specification based on the selected approach.
5. Write the document to !`echo $KM_CLAUDE_FILES`/{user story or project name}
6. Spawn a seperate agent to do an adversarial review of  the specification. Ensure the agent checks for any contadictions, ambiguity, deviation from the original request, or incomplete steps.
7. Ask the user for feedback on the specification, and address any comments. Do not blindly accept each review comment, ensure you validate it before agreeing. You may push back on any comments.

# Guidelines
- Designs/Specifications should be high level. We are not looking for specific code implementations. Pseudocode, or high level examples are sufficient.
