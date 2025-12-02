---
description: Analyzes the session to identify how the initial prompt could have been improved based on subsequent conversations and work performed
---

You are analyzing a Claude Code session to provide actionable feedback on the initial prompt.

# Analysis Framework

## 1. Context Gap Analysis
- What context was missing from the initial prompt that was revealed through follow-up questions?
- What assumptions did Claude make that were later corrected?
- What files, directories, or system details should have been specified upfront?

## 2. Scope Clarity Issues
- Was the scope too vague or too broad?
- What specific deliverables were unclear initially?
- What edge cases or constraints emerged later that should have been mentioned?

## 3. Technical Specificity
- What technical details (frameworks, versions, patterns, standards) were clarified later?
- What implementation preferences were revealed through iteration?
- What performance, security, or quality requirements were implicit?

## 4. Communication Efficiency
- How many clarification rounds could have been avoided?
- What examples or references would have accelerated understanding?
- What output format expectations were unclear?

# Output Format

Provide your analysis in this structure:

## Session Overview
- Brief summary of what was accomplished
- Number of messages/iterations required

## Initial Prompt Analysis
```
[Quote the original prompt]
```

## Key Issues Identified
1. **[Issue Category]**: [Specific problem]
2. **[Issue Category]**: [Specific problem]
   ...

## Improved Prompt
```
[Rewritten version of the initial prompt incorporating all learnings]
```

## Improvement Patterns
- **Added**: [What was added and why]
- **Clarified**: [What was made more specific]
- **Structured**: [How organization improved clarity]

## Transferable Lessons
[2-3 general principles the user can apply to future prompts]

## CLAUDE.md Recommendations

Based on this session, consider adding the following to your project's CLAUDE.md files:

### Suggested CLAUDE.md Location: `[path/to/CLAUDE.md]`
```markdown
[Specific content that should be added to help with future sessions]
```

**Why this helps**: [Explanation of how this context would have improved this session]

### Additional CLAUDE.md Files to Consider
- **`[suggested/path/CLAUDE.md]`**: [What type of context should go here and why]

# Guidelines

- Be specific: cite actual examples from the conversation
- Be actionable: suggest concrete additions/changes
- Be honest: if the initial prompt was good, say so and suggest minor refinements
- Focus on what would have saved time or improved outcomes
- Consider both what was said and what was done (code changes, files created, etc.)
- Don't just list what was missing—explain why it mattered

# Important

Analyze the ENTIRE conversation history to understand the full arc from initial request to final implementation. Pay special attention to:
- Questions Claude asked
- User corrections or clarifications
- Iterations on the same code/feature
- Scope changes or additions
- Technical decisions that weren't specified upfront
