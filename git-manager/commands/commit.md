---
description: "Commit operations: add, commit, status, diff"
argument-hint: "[add|commit|status|diff] [files...]"
allowed-tools: Bash(git add:*), Bash(git commit:*), Bash(git status:*), Bash(git diff:*)
---

# /git:commit

## Context

- Current git status: !`git status`
- Current git diff (staged and unstaged): !`git diff HEAD`
- Current branch: !`git branch --show-current`
- Recent commits: !`git log --oneline -5`

## Sub-command Routing

Parse `$ARGUMENTS` to determine sub-command:

- **No arguments** or **commit**: Full commit flow (status → stage all → commit with generated message)
- **add**: Stage specified files or all changes
- **status**: Show git status only
- **diff**: Show diff (staged if `--staged` flag, otherwise unstaged)

## Sub-command: commit (default)

1. Run `git status` to see changes
2. Run `git diff HEAD` to understand what changed
3. Stage all changes: `git add -A`
4. Generate a conventional commit message based on the diff
5. Commit: `git commit -m "<message>"`
6. Do not use any other tools. Do all steps in a single message.

## Sub-command: add

1. If specific files provided in `$ARGUMENTS`: `git add <files>`
2. If no files: `git add -A`
3. Report staged files

## Sub-command: status

1. Run `git status`
2. Display the output

## Sub-command: diff

1. If `--staged` in `$ARGUMENTS`: run `git diff --staged`
2. If `--stat` in `$ARGUMENTS`: run `git diff --stat`
3. Otherwise: run `git diff`
4. Display the output
