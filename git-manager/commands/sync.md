---
description: "Sync operations: pull, push, fetch, rebase"
argument-hint: "[pull|push|fetch|rebase] [args]"
allowed-tools: Bash(git pull:*), Bash(git push:*), Bash(git fetch:*), Bash(git rebase:*)
---

# /git:sync

## Context

- Current branch: !`git branch --show-current`
- Remote status: !`git remote -v`
- Tracking info: !`git branch -vv`

## Sub-command Routing

Parse `$ARGUMENTS`:

- **No arguments**: Pull with rebase (default sync behavior)
- **pull**: Pull from remote
- **push**: Push to remote
- **fetch**: Fetch remote references
- **rebase**: Rebase onto target branch

## Sub-command: pull (default)

1. Build command: `git pull`
2. If `--rebase` in `$ARGUMENTS` or default: add `--rebase`
3. If `--autostash` in `$ARGUMENTS`: add `--autostash`
4. Execute and report result

## Sub-command: push

1. Check if branch has upstream: `git rev-parse --abbrev-ref @{upstream} 2>/dev/null`
2. If no upstream or `--set-upstream`: `git push -u origin <branch>`
3. If `--force`: `git push --force-with-lease origin <branch>`
4. Otherwise: `git push`
5. Report result

## Sub-command: fetch

1. If `--all`: `git fetch --all`
2. If `--prune`: `git fetch --prune`
3. Otherwise: `git fetch`
4. Report fetched references

## Sub-command: rebase

1. Extract target branch from `$ARGUMENTS`
2. If `--interactive`: `git rebase -i <target>`
3. Otherwise: `git rebase <target>`
4. Report result
