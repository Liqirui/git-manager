---
description: "Issue management: list, create, view, close, comment, label"
argument-hint: "[list|create|view|close|comment|label] [number] [args]"
allowed-tools: Bash(gh issue:*), Bash(git remote:*), Read
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

### Tool: gh CLI only (MCP has no issue write tool)

1. Extract title from `--title` in `$ARGUMENTS`
2. Extract body from `--body`
3. Extract labels from `--label`
4. Run: `gh issue create --title "<title>" --body "<body>" --label "<label>"`
5. Report created issue

## Sub-command: view

### MCP path:
1. Call `issue_read` with method `get`
2. Also call `issue_read` with method `get_comments`

### gh fallback:
1. `gh issue view <number> --comments`

## Sub-command: close

### Tool: gh CLI only (MCP has no issue write tool)

1. Extract issue number from `$ARGUMENTS`
2. Extract reason from `--reason` (default: completed)
3. Run: `gh issue close <number> --reason <reason>`
4. Report success

## Sub-command: comment

### MCP path:
1. Call `add_issue_comment` with body from `--body`

### gh fallback:
1. `gh issue comment <number> --body "<body>"`

## Sub-command: label

### Tool: gh CLI only (MCP has no issue write tool)

1. Extract issue number from `$ARGUMENTS`
2. If `--add`: `gh issue edit <number> --add-label <label>`
3. If `--remove`: `gh issue edit <number> --remove-label <label>`
4. Report result
