---
name: setup
description: Setup the users machine to use the dev skills in this plugin
user-invocable: true
disable-model-invocation: true
---

# Environment Variables
The following environment variables are referenced in the skills in this plugin.
- KM_CLAUDE_FILES - Where the created documents (plans, specs, research, etc) are stored. Can be a relative path or a global path.

The variables should be set in a ~/.kmrc file and it must be sourced in the users ~/.zshrc

# Current variable values
KM_CLAUDE_FILES = !`echo KM_CLAUDE_FILES`

# Process
1. Check that ~/.kmrc file exists. If it doesn't exist, create it
2. Grep ~/.zshrc to check if ~/.kmrc is sourced, if not add it.
3. Identify any unset envrionment variables
4. For each unset environment variable, prompt the user for what they would like to set it to, and set the value in ~/.kmrc
