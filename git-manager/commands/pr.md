---
description: "Pull request operations: create, list, view, merge, review, diff, comment"
argument-hint: "[create|list|view|merge|review|diff|comment] [number] [args]"
allowed-tools: Bash, Glob, Grep, Read
---

# /git:pr

## Context

- Current branch: !`git branch --show-current`
- Remote info: !`git remote -v`

## Tool Routing

**Preferred:** GitHub MCP tools (mcp__plugin_github_github__*)
**Fallback:** `gh` CLI commands

For each sub-command, try MCP first. If MCP tool is not available or returns error, fall back to gh CLI.

## Sub-command Routing

Parse `$ARGUMENTS`:

- **No arguments**: List open PRs for current repo
- **create**: Create PR from current branch
- **list**: List PRs with optional filters
- **view \<number\>**: View PR details
- **merge \<number\>**: Merge PR
- **review \<number\>**: Submit review
- **diff \<number\>**: Show PR diff
- **comment \<number\>**: Add comment

## Sub-command: create

### MCP path:
1. Get current branch and repo info from git remote
2. Call `create_pull_request` with:
   - owner/repo from git remote
   - head: current branch
   - base: default branch (usually main/master)
   - title: from `$ARGUMENTS` or auto-generate from recent commits
   - body: auto-generate from commits
   - draft: if `--draft` flag

### gh fallback:
1. `gh pr create --title "<title>" --body "<body>"`
2. If `--draft`: add `--draft` flag

## Sub-command: list (default)

### MCP path:
1. Call `list_pull_requests` with:
   - state: from `--state` (default: OPEN)
   - owner/repo from git remote

### gh fallback:
1. `gh pr list --state <state> --json number,title,author,state`

## Sub-command: view

### MCP path:
1. Call `pull_request_read` with method `get`
2. Include diff if `--diff` flag

### gh fallback:
1. `gh pr view <number>`
2. If `--diff`: `gh pr diff <number>`

## Sub-command: merge

### MCP path:
1. Call `merge_pull_request` with:
   - merge_method: from `--method` (default: squash)

### gh fallback:
1. `gh pr merge <number> --squash` (or --rebase/--merge)

## Sub-command: review

### Tool: gh CLI only (MCP has no review write tool)

1. Extract event from `--event` (default: comment)
2. Extract body from `--body` or auto-generate
3. Run: `gh pr review <number> --<event> --body "<body>"`
4. Report result

## Sub-command: diff

### MCP path:
1. Call `pull_request_read` with method `get_diff`

### gh fallback:
1. `gh pr diff <number>`

## Sub-command: comment

### MCP path:
1. Call `add_issue_comment` with body from `--body`

### gh fallback:
1. `gh pr comment <number> --body "<body>"`
