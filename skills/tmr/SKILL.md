---
name: tmr
description: Run shell commands through tmux in the active session with strict pane management, output capture, and send-keys workflow. Use when a task requires running commands via tmux panes, checking pane status, or splitting panes in the active session.
---

# Tmr

## Overview

Use tmux to run every command inside the active session while capturing output for verification.

## Workflow

### 1) Resolve session

- If already inside tmux, get the session name: `tmux display-message -p '#{session_name}'`.
- If not inside tmux, list sessions: `tmux list-sessions -F '#{session_name}'` and choose the current one (ask the user if unclear).
- Use `<session>` in all subsequent tmux commands.

### 2) Choose a pane

- If reusing the active pane, get it directly: `tmux display-message -p -t <session> '#{pane_index}'`.
- If you need a new pane, create it and capture its index: `tmux split-window -h -t <session> -P -F '#{pane_index}'`.
- If you need to inspect all panes, list them: `tmux list-panes -t <session> -F '#{pane_index}'`.
- Avoid any reserved pane noted by the environment or user instructions.

### 3) Check pane state

- Capture the pane before sending commands: `tmux capture-pane -pt <session>.<pane>`.

### 4) Send commands

- Prefer a single call without `-l`: `tmux send-keys -t <session>.<pane> "cmd" C-m`.
- If you need `-l`, use two calls: `tmux send-keys -t <session>.<pane> -l 'cmd'` then `tmux send-keys -t <session>.<pane> C-m`.
- Use `-l` only when sending literal text that includes special key names (like `C-m`, `C-c`) or when you must prevent tmux from interpreting key names.
- Example without `-l`: `tmux send-keys -t <session>.<pane> "ls -la" C-m`.
- Example with `-l`: `tmux send-keys -t <session>.<pane> -l 'printf \"done\\n\"'` then `tmux send-keys -t <session>.<pane> C-m`.

### 5) Read output

- Capture the pane after execution with `tmux capture-pane -pt <session>.<pane>`.
- Use `tmux capture-pane -pt <session>.<pane> -S -200` when output might be scrolled.

## Conventions

- Keep all shell commands inside tmux panes.
- Resolve the session once per task and reuse it consistently.
- Prefer pane-creation that returns the new index (`-P -F '#{pane_index}'`) to avoid extra listing.
- Reuse existing panes when possible.
- Capture output instead of assuming success.
