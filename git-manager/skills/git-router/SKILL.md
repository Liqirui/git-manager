---
name: git-router
description: "Use when user mentions git, GitHub, PR, pull request, issue, branch, commit, release, CI/CD, workflow, merge, clone, fork, rebase, push, pull, fetch, or any version control operation. Routes to the correct /git:* command. ALWAYS use this skill before answering git-related questions."
---

# Git Router

You are the routing layer for the git-manager plugin. Your job is to understand the user's intent and guide them to the right `/git:*` command.

## How to Route

1. Parse the user's intent from their message
2. Match to the appropriate command and sub-command
3. Present the recommendation with a usage example

## Intent → Command Mapping

| User Intent | Command | Example |
|-------------|---------|---------|
| "提交代码" / "commit" | `/git:commit` | `/git:commit` |
| "创建分支" / "switch branch" | `/git:branch` | `/git:branch create feature-x` |
| "推送" / "拉取" / "同步" | `/git:sync` | `/git:sync push` |
| "创建PR" / "合并请求" | `/git:pr` | `/git:pr create` |
| "查看PR" / "PR状态" | `/git:pr` | `/git:pr list` |
| "提issue" / "问题管理" | `/git:issue` | `/git:issue create --title "bug"` |
| "CI状态" / "workflow" | `/git:ci` | `/git:ci status` |
| "发版" / "release" | `/git:release` | `/git:release create v1.0.0` |
| "创建仓库" / "fork" | `/git:repo` | `/git:repo create my-project` |
| "清理分支" | `/git:branch` | `/git:branch clean` |

## Response Format

When routing, respond with:

```
建议使用: /git:<command> <sub-command> [args]

示例: /git:<command> <sub-command> <example-args>
```

If the user's intent is ambiguous, ask them to clarify using AskUserQuestion with 2-3 options.
