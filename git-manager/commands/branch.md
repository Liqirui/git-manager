---
description: "Branch operations: list, create, delete, switch, clean"
argument-hint: "[list|create|delete|switch|clean] [name]"
allowed-tools: Bash(git branch:*), Bash(git checkout:*), Bash(git worktree:*), Bash(git rev-parse:*)
---

# /git:branch

## Context

- Current branch: !`git branch --show-current`
- Local branches: !`git branch -v`
- Remote branches: !`git branch -r`

## Sub-command Routing

Parse `$ARGUMENTS`:

- **No arguments**: List local branches
- **list**: List branches (add `--all` for remote, `--merged` for merged)
- **create \<name\>**: Create and switch to new branch
- **delete \<name\>**: Delete branch locally (add `--remote` for remote too)
- **switch \<name\>**: Switch to branch
- **clean**: Remove branches marked [gone]

## Sub-command: list (default)

1. Run `git branch -v` for local branches
2. If `--all` in `$ARGUMENTS`: also run `git branch -r` for remote
3. If `--merged` in `$ARGUMENTS`: run `git branch --merged`
4. Display formatted output

## Sub-command: create

1. Extract branch name from `$ARGUMENTS`
2. If `--from <branch>` specified: `git checkout -b <name> <from-branch>`
3. Otherwise: `git checkout -b <name>`
4. Report success

## Sub-command: delete

1. Extract branch name from `$ARGUMENTS`
2. Confirm with user before deleting (destructive operation)
3. Check if branch has associated worktree: `git worktree list`
4. If worktree exists: `git worktree remove --force <worktree-path>`
5. Delete local branch: `git branch -D <name>`
6. If `--remote` in `$ARGUMENTS`: `git push origin --delete <name>`

## Sub-command: switch

1. Extract branch name from `$ARGUMENTS`
2. Run `git checkout <name>`
3. Report current branch

## Sub-command: clean

1. Run `git branch -v` to find [gone] branches
2. List all [gone] branches to the user
3. If `--dry-run` in `$ARGUMENTS`: only list, don't delete, then stop
4. Confirm with user before deleting (batch destructive operation)
5. For each [gone] branch:
   - Check worktree: `git worktree list`
   - Remove worktree if exists: `git worktree remove --force <path>`
   - Delete branch: `git branch -D <name>`
6. Report cleaned branches
