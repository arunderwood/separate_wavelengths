---
title: Sharing Claude Code Project Configs Across Claude Squad Sessions
date: 2025-12-29 17:22:52
tags: Claude Code, Claude Squad, Git, Worktrees
---

I've been using [Claude Code](https://docs.anthropic.com/en/docs/claude-code), Anthropic's CLI for AI-assisted coding, to help with a side project. Recently I started running multiple sessions in parallel using [claude-squad](https://github.com/smtg-ai/claude-squad), which spins up isolated instances via [git worktrees](https://git-scm.com/docs/git-worktree). It's a great workflow, but I ran into a minor annoyance.

## The Problem

When using claude-squad, each worktree gets its own `.claude/` directory. Claude Code stores per-project permissions in `.claude/settings.local.json` - this file stays in `.gitignore` since it contains local preferences rather than project configuration.

The problem: every time claude-squad creates a new worktree, you start with a fresh `settings.local.json`. Any permissions you've granted (like allowing `./gradlew build` to run without prompting) need to be re-approved in each session.

```
$ git worktree list
/Users/youruser/checkouts/myproject                           b44b5eb [main]
/Users/youruser/.claude-squad/worktrees/feature-branch        bd9bc1a [feature-branch]
/Users/youruser/.claude-squad/worktrees/bugfix                a1c2d3e [bugfix]
```

Three worktrees means three separate permission configs. Tedious.

## The Solution

Git's `post-checkout` hook fires when a new worktree is created. By detecting when the previous HEAD is null (all zeros), we can identify new worktree creation and automatically symlink the settings file back to the main worktree.

### Create the hook

Add this script to `.git/hooks/post-checkout`:

```bash
#!/bin/bash

# post-checkout hook: Symlink shared Claude settings for new worktrees
#
# When a new worktree is created via `git worktree add`, this hook
# creates a symlink to the main worktree's .claude/settings.local.json
# so all Claude Code instances share the same permissions config.

PREV_HEAD="$1"
NEW_HEAD="$2"
BRANCH_CHECKOUT="$3"

# Detect new worktree: previous HEAD is null (all zeros)
NULL_REF="0000000000000000000000000000000000000000"

if [ "$PREV_HEAD" = "$NULL_REF" ]; then
    # This is a new worktree or clone
    MAIN_WORKTREE="$HOME/checkouts/myproject"
    SHARED_SETTINGS="$MAIN_WORKTREE/.claude/settings.local.json"
    LOCAL_CLAUDE_DIR=".claude"
    LOCAL_SETTINGS="$LOCAL_CLAUDE_DIR/settings.local.json"

    # Only proceed if we're in a worktree (not the main worktree itself)
    CURRENT_DIR="$(pwd)"
    if [ "$CURRENT_DIR" != "$MAIN_WORKTREE" ] && [ -f "$SHARED_SETTINGS" ]; then
        mkdir -p "$LOCAL_CLAUDE_DIR"

        # Remove existing file if present (not a symlink)
        if [ -f "$LOCAL_SETTINGS" ] && [ ! -L "$LOCAL_SETTINGS" ]; then
            rm "$LOCAL_SETTINGS"
        fi

        # Create symlink if not already present
        if [ ! -L "$LOCAL_SETTINGS" ]; then
            ln -s "$SHARED_SETTINGS" "$LOCAL_SETTINGS"
            echo "Created symlink: $LOCAL_SETTINGS -> $SHARED_SETTINGS"
        fi
    fi
fi
```

Update `MAIN_WORKTREE` to point to your primary checkout location.

### Make it executable

```bash
chmod +x .git/hooks/post-checkout
```

### Fix existing worktrees (optional)

For worktrees that already exist, create the symlink manually:

```bash
mkdir -p .claude
ln -sf ~/checkouts/myproject/.claude/settings.local.json .claude/settings.local.json
```

## Verification

Create a new worktree and check for the symlink:

```bash
$ git worktree add ../test-worktree main
Created symlink: .claude/settings.local.json -> /Users/youruser/checkouts/myproject/.claude/settings.local.json

$ ls -la ../test-worktree/.claude/settings.local.json
lrwxr-xr-x  1 youruser  staff  69 Dec 29 16:57 .claude/settings.local.json -> /Users/youruser/checkouts/myproject/.claude/settings.local.json

$ git worktree remove ../test-worktree
```

To verify Claude Code is actually reading from the symlink, you can add a test permission to the main settings file and confirm it works in the worktree session without prompting.

## Result

All worktrees now automatically share the same Claude Code permissions. Grant a permission once in any session, and it applies everywhere. The hook is idempotent (safe to run multiple times) and non-destructive (won't overwrite existing non-symlink files without explicit removal).

One less thing to think about when spinning up parallel Claude sessions.
