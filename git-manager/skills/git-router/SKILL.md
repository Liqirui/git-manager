---
name: git-router
description: "Use when user wants to PERFORM a git/GitHub operation: commit, push, pull, create PR, merge, open issue, check CI, release, fork, clone, branch management, or sync. Trigger on action requests like 'commit this', 'create a PR', 'push changes', 'check CI status'. SKIP when user is reading/explaining code, doing code review, or when git terms appear in code context (e.g. 'this commit function has a bug')."
---

# Git Router

You are the routing layer for the git-manager plugin. Your job is to understand the user's intent and guide them to the right `/git:*` command.

## When to Route

Route ONLY when the user wants to execute a git/GitHub operation. Examples:

- "帮我提交代码" → `/git:commit`
- "创建一个 PR" → `/git:pr create`
- "推送一下" → `/git:sync push`
- "CI 怎么样了" → `/git:ci status`

Do NOT route when:

- User is asking about code that happens to involve git concepts
- User is doing code review (even if reviewing git-related code)
- User is explaining or learning about git
- Git terms appear in code context (variable names, function names, etc.)

## Intent → Command Mapping

| User Intent | Command | Example |
|-------------|---------|---------|
| "提交代码" / "commit these changes" | `/git:commit` | `/git:commit` |
| "创建分支" / "switch to feature-x" | `/git:branch` | `/git:branch create feature-x` |
| "推送" / "拉取" / "同步到远程" | `/git:sync` | `/git:sync push` |
| "创建PR" / "开个合并请求" | `/git:pr` | `/git:pr create` |
| "查看PR" / "PR 列表" | `/git:pr` | `/git:pr list` |
| "提 issue" / "报个 bug" | `/git:issue` | `/git:issue create --title "bug"` |
| "CI 状态" / "workflow 跑完了吗" | `/git:ci` | `/git:ci status` |
| "发版" / "创建 release" | `/git:release` | `/git:release create v1.0.0` |
| "创建仓库" / "fork 一下" | `/git:repo` | `/git:repo create my-project` |
| "清理无用分支" | `/git:branch` | `/git:branch clean` |
| "环境检查" / "插件配置" | `/git:setup` | `/git:setup` |

## Response Format

When routing, respond with:

```
建议使用: /git:<command> <sub-command> [args]

示例: /git:<command> <sub-command> <example-args>
```

If the user's intent is ambiguous, ask them to clarify using AskUserQuestion with 2-3 options.
