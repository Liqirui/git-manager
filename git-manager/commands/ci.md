---
description: "CI/CD operations: status, rerun, logs, watch"
argument-hint: "[status|rerun|logs|watch] [run-id]"
allowed-tools: Bash(gh run:*), Bash(gh workflow:*), Read
---

# /git:ci

## Tool Routing

All CI/CD operations use `gh` CLI (MCP does not support workflow management).

## Sub-command Routing

Parse `$ARGUMENTS`:

- **No arguments**: Show latest workflow status
- **status**: List recent workflow runs
- **rerun \<run-id\>**: Rerun a workflow
- **logs \<run-id\>**: View workflow logs
- **watch \<run-id\>**: Watch workflow in real-time

## Sub-command: status (default)

1. Run: `gh run list --limit 10`
2. Display formatted table of recent runs

## Sub-command: rerun

1. Extract run ID from `$ARGUMENTS`
2. If `--failed-only`: `gh run rerun <id> --failed-only`
3. Otherwise: `gh run rerun <id>`
4. Report success

## Sub-command: logs

1. Extract run ID from `$ARGUMENTS`
2. If `--job`: `gh run view <id> --log --job <job-id>`
3. Otherwise: `gh run view <id> --log`
4. Display logs

## Sub-command: watch

1. Extract run ID from `$ARGUMENTS`
2. Run: `gh run watch <id>`
3. Stream output until completion
