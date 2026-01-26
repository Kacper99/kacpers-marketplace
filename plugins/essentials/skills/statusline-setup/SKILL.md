---
name: statusline-setup
description: Configure Claude Code's statusline with a Powerlevel10k-inspired theme showing directory, git branch, lines changed, and context usage progress bar.
---

# Statusline Configuration Skill

## Overview

This skill configures Claude Code's statusline with a Powerlevel10k-inspired theme that displays:
- **Shortened directory** - Shows `~/currentDir` format
- **Git branch** - Displays branch name with `` icon
- **Lines changed** - Shows additions (green `+N`) and deletions (red `-N`)
- **Context usage** - Progress bar with gradient shading (green/yellow/red)

## Pre-Installation Checks

Before installing, you MUST check for existing configurations:

### 1. Check for existing statusline configuration

Read `~/.claude/settings.json` and check if a `statusLine` key already exists.

### 2. Check for existing statusline script

Check if `~/.claude/statusline-command.sh` already exists.

### 3. Handle existing configuration

**If either an existing `statusLine` configuration OR an existing script is found:**

Use the `AskUserQuestion` tool to ask the user:

```
Question: "You already have a statusline configuration. Do you want to replace it with the Powerlevel10k-inspired theme?"
Options:
  - "Yes, replace it" - Proceed with installation
  - "No, keep my current setup" - Abort installation
```

**If the user chooses to replace AND an existing script file exists at a different path:**

Use the `AskUserQuestion` tool to ask:

```
Question: "Should I delete your old statusline script?"
Options:
  - "Yes, delete it" - Delete the old script file
  - "No, keep it" - Leave the old script file in place
```

**If the user chooses not to replace:** Stop here and inform the user that the existing configuration has been preserved.

---

## Installation Instructions

Follow these steps to configure the statusline:

### Step 1: Write the statusline script

Write the following bash script to `~/.claude/statusline-command.sh`:

```bash
#!/bin/bash

# Powerlevel10k-inspired statusline for Claude Code
# Displays: directory | git branch | lines changed | context progress bar

# Colors
RESET="\033[0m"
BOLD="\033[1m"
DIM="\033[2m"
BLUE="\033[34m"
GREEN="\033[32m"
RED="\033[31m"
YELLOW="\033[33m"
CYAN="\033[36m"
MAGENTA="\033[35m"

# Get shortened directory (~/Projects/my-app -> ~/my-app)
get_short_dir() {
    local dir="$PWD"
    if [[ "$dir" == "$HOME"* ]]; then
        dir="~${dir#$HOME}"
    fi
    # Only show home prefix and current directory
    local current=$(basename "$dir")
    if [[ "$dir" == "~" ]]; then
        echo "~"
    elif [[ "$dir" == "~/"* ]]; then
        echo "~/$current"
    else
        echo "/$current"
    fi
}

# Get git branch
get_git_branch() {
    local branch=$(git symbolic-ref --short HEAD 2>/dev/null || git rev-parse --short HEAD 2>/dev/null)
    if [[ -n "$branch" ]]; then
        echo " $branch"
    fi
}

# Get git diff stats
get_git_stats() {
    if git rev-parse --git-dir > /dev/null 2>&1; then
        local stats=$(git diff --shortstat 2>/dev/null)
        local staged_stats=$(git diff --cached --shortstat 2>/dev/null)

        local insertions=0
        local deletions=0

        # Parse unstaged changes
        if [[ -n "$stats" ]]; then
            local ins=$(echo "$stats" | grep -oE '[0-9]+ insertion' | grep -oE '[0-9]+')
            local del=$(echo "$stats" | grep -oE '[0-9]+ deletion' | grep -oE '[0-9]+')
            insertions=$((insertions + ${ins:-0}))
            deletions=$((deletions + ${del:-0}))
        fi

        # Parse staged changes
        if [[ -n "$staged_stats" ]]; then
            local ins=$(echo "$staged_stats" | grep -oE '[0-9]+ insertion' | grep -oE '[0-9]+')
            local del=$(echo "$staged_stats" | grep -oE '[0-9]+ deletion' | grep -oE '[0-9]+')
            insertions=$((insertions + ${ins:-0}))
            deletions=$((deletions + ${del:-0}))
        fi

        local output=""
        if [[ $insertions -gt 0 ]]; then
            output="${GREEN}+${insertions}${RESET}"
        fi
        if [[ $deletions -gt 0 ]]; then
            [[ -n "$output" ]] && output="$output "
            output="${output}${RED}-${deletions}${RESET}"
        fi
        echo -e "$output"
    fi
}

# Build context progress bar
# Usage: CLAUDE_CONTEXT_USED and CLAUDE_CONTEXT_TOTAL env vars
build_progress_bar() {
    local used="${CLAUDE_CONTEXT_USED:-0}"
    local total="${CLAUDE_CONTEXT_TOTAL:-200000}"

    if [[ "$total" -eq 0 ]]; then
        return
    fi

    local percentage=$((used * 100 / total))
    local bar_width=10
    local filled=$((percentage * bar_width / 100))
    local empty=$((bar_width - filled))

    # Choose color based on percentage
    local color="$GREEN"
    if [[ $percentage -ge 75 ]]; then
        color="$RED"
    elif [[ $percentage -ge 50 ]]; then
        color="$YELLOW"
    fi

    # Build the bar
    local bar=""
    for ((i=0; i<filled; i++)); do
        bar+="█"
    done
    for ((i=0; i<empty; i++)); do
        bar+="░"
    done

    echo -e "${DIM}[${RESET}${color}${bar}${RESET}${DIM}]${RESET} ${percentage}%"
}

# Main output
main() {
    local dir=$(get_short_dir)
    local branch=$(get_git_branch)
    local stats=$(get_git_stats)
    local progress=$(build_progress_bar)

    local output=""

    # Directory
    output="${BLUE}${BOLD}${dir}${RESET}"

    # Git branch
    if [[ -n "$branch" ]]; then
        output="${output} ${MAGENTA}${branch}${RESET}"
    fi

    # Git stats
    if [[ -n "$stats" ]]; then
        output="${output} ${stats}"
    fi

    # Context progress
    if [[ -n "$progress" ]]; then
        output="${output}  ${progress}"
    fi

    echo -e "$output"
}

main
```

### Step 2: Make the script executable

```bash
chmod +x ~/.claude/statusline-command.sh
```

### Step 3: Update settings.json

Add or merge the following configuration into `~/.claude/settings.json`:

```json
{
  "statusLine": {
    "type": "command",
    "command": "~/.claude/statusline-command.sh"
  }
}
```

If the file already exists, merge the `statusLine` key into the existing configuration.

## Verification

After installation:
1. Restart Claude Code
2. The statusline should appear at the bottom showing:
   - Current directory in blue
   - Git branch with  icon in magenta (if in a git repo)
   - Lines added/removed in green/red (if there are changes)
   - Context usage progress bar

## Customization

You can modify `~/.claude/statusline-command.sh` to customize:
- Colors (change ANSI codes at the top)
- Progress bar width (change `bar_width` variable)
- Progress bar characters (change `█` and `░`)
- Directory format (modify `get_short_dir` function)
