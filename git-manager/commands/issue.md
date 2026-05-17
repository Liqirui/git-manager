---
description: "Issue management: list, create, view, close, comment, label"
argument-hint: "[list|create|view|close|comment|label] [number] [args]"
allowed-tools: Bash, Glob, Grep, Read
---

# /git:issue

## Context

- Current repo: !`git remote -v | head -1`

## Tool Routing

**Preferred:** GitHub MCP tools
**Fallback:** `gh` CLI commands

## Sub-command Routing

Parse `$ARGUMENTS`:

- **No arguments**: List open issues
- **list**: List issues with filters
- **create**: Create new issue
- **view \<number\>**: View issue details
- **close \<number\>**: Close issue
- **comment \<number\>**: Add comment
- **label \<number\>**: Manage labels

## Sub-command: list (default)

### MCP path:
1. Call `list_issues` with:
   - state: from `--state` (default: OPEN)
   - labels: from `--label`
   - owner/repo from git remote

### gh fallback:
1. `gh issue list --state <state> --label <label>`

## Sub-command: create

### MCP path:
1. Call `issue_write` with method `create`:
   - title: from `--title` in `$ARGUMENTS`
   - body: from `--body`
   - labels: from `--label`

### gh fallback:
1. `gh issue create --title "<title>" --body "<body>" --label "<label>"`

## Sub-command: view

### MCP path:
1. Call `issue_read` with method `get`
2. Also call `issue_read` with method `get_comments`

### gh fallback:
1. `gh issue view <number> --comments`

## Sub-command: close

### MCP path:
1. Call `issue_write` with method `update`:
   - state: "closed"
   - state_reason: from `--reason` (default: completed)

### gh fallback:
1. `gh issue close <number> --reason <reason>`

## Sub-command: comment

### MCP path:
1. Call `add_issue_comment` with body from `--body`

### gh fallback:
1. `gh issue comment <number> --body "<body>"`

## Sub-command: label

### MCP path:
1. If `--add`: update issue to add label
2. If `--remove`: update issue to remove label

### gh fallback:
1. If `--add`: `gh issue edit <number> --add-label <label>`
2. If `--remove`: `gh issue edit <number> --remove-label <label>`
