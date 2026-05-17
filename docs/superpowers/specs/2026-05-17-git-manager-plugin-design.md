# git-manager 插件设计规格

## 概述

创建一个名为 `git-manager` 的 Claude Code 插件，整合 GitHub MCP Server 和 GitHub CLI (`gh`) 的能力，提供统一的 Git/GitHub 管理体验。

**核心目标：**
- 统一本地 git 操作和 GitHub 远程操作
- 智能路由：根据操作类型自动选择 MCP 或 gh CLI
- 替代现有 `commit-commands` 插件，提供更完整的功能覆盖

## 插件结构

```
git-manager/
├── .claude-plugin/
│   └── plugin.json
├── .mcp.json
├── commands/
│   ├── pr.md
│   ├── issue.md
│   ├── ci.md
│   ├── release.md
│   ├── repo.md
│   ├── branch.md
│   ├── commit.md
│   └── sync.md
├── skills/
│   └── git-router/
│       └── SKILL.md
├── agents/
│   └── smart-router.md
└── references/
    ├── mcp-tools.md
    ├── gh-cli-ops.md
    └── routing-rules.md
```

## 智能路由规则

### 路由矩阵

| 操作类别 | 优先工具 | 降级工具 | 原因 |
|---------|---------|---------|------|
| PR 创建/查看/合并/审查 | MCP | gh CLI | 结构化返回 |
| Issue 创建/查看/评论 | MCP | gh CLI | 结构化返回 |
| 代码搜索 | MCP | gh CLI | search_code 端到端 |
| 文件读写 | MCP | gh CLI | get_file_contents |
| 仓库搜索/创建 | MCP | gh CLI | 结构化数据 |
| 分支/标签列表 | MCP | git | 结构化返回 |
| PR review 评论 | MCP | gh CLI | pull_request_review_write |
| Secret scanning | MCP | 无 | MCP 独有 |
| CI/CD workflow 状态 | gh CLI | 无 | MCP 不支持 |
| CI/CD 重跑/取消 | gh CLI | 无 | MCP 不支持 |
| Release 管理 | gh CLI | MCP | gh 更完整 |
| Secrets/Variables | gh CLI | 无 | MCP 不支持 |
| 任意 GitHub API | gh CLI | 无 | gh api 无限制 |
| 本地 git 操作 | git 原生 | 无 | 纯本地 |

### 降级策略

1. 首选工具不可用时（连接失败/未认证），自动降级到备选，无需用户确认
2. 降级时在输出中提示："MCP 不可用，已使用 gh CLI 完成操作"
3. 两个都不可用时，提示用户配置并中止操作
4. 对于写操作（create/delete/merge），降级后再次确认操作结果，确保一致性

## 命令详细设计

### 1. /git:pr

**Frontmatter:**
```yaml
description: "Pull request operations: create, list, view, merge, review, diff, comment"
argument-hint: "[create|list|view|merge|review|diff|comment] [args]"
allowed-tools: ["Bash", "Glob", "Grep", "Read"]
```

**子操作：**

| 子操作 | 路由 | 说明 | 参数 |
|--------|------|------|------|
| `create` | MCP → gh | 从当前分支创建 PR | [--title] [--body] [--base] [--draft] |
| `list` | MCP | 列出 PR | [--state open/closed/all] [--author] [--label] |
| `view` | MCP | 查看 PR 详情 | <pr-number> |
| `merge` | MCP | 合并 PR | <pr-number> [--method squash/rebase/merge] |
| `review` | MCP | 提交审查 | <pr-number> [--event approve/request_changes/comment] [--body] |
| `diff` | MCP → gh | 查看 PR diff | <pr-number> |
| `comment` | MCP | 添加评论 | <pr-number> [--body] |

**默认行为：** 无参数时显示当前分支关联的 PR 列表。

### 2. /git:issue

**Frontmatter:**
```yaml
description: "Issue management: list, create, view, close, comment, label"
argument-hint: "[list|create|view|close|comment|label] [args]"
allowed-tools: ["Bash", "Glob", "Grep", "Read"]
```

**子操作：**

| 子操作 | 路由 | 说明 | 参数 |
|--------|------|------|------|
| `list` | MCP | 列出 issue | [--state open/closed] [--label] [--author] |
| `create` | MCP | 创建 issue | --title [--body] [--label] [--assignee] |
| `view` | MCP | 查看详情 | <issue-number> |
| `close` | MCP | 关闭 issue | <issue-number> [--reason completed/not_planned/duplicate] |
| `comment` | MCP | 添加评论 | <issue-number> --body |
| `label` | MCP | 管理标签 | <issue-number> [--add/--remove] <label> |

**默认行为：** 无参数时列出当前仓库的 open issues。

### 3. /git:ci

**Frontmatter:**
```yaml
description: "CI/CD operations: status, rerun, logs, watch"
argument-hint: "[status|rerun|logs|watch] [args]"
allowed-tools: ["Bash", "Read"]
```

**子操作：**

| 子操作 | 路由 | 说明 | 参数 |
|--------|------|------|------|
| `status` | gh CLI | 查看 workflow 状态 | [--limit N] |
| `rerun` | gh CLI | 重跑 workflow | <run-id> [--failed-only] |
| `logs` | gh CLI | 查看日志 | <run-id> [--job] [--step] |
| `watch` | gh CLI | 实时监控 | <run-id> |

**默认行为：** 无参数时显示当前分支最新 workflow 状态。

### 4. /git:release

**Frontmatter:**
```yaml
description: "Release management: list, create, view, delete"
argument-hint: "[list|create|view|delete] [args]"
allowed-tools: ["Bash", "Read"]
```

**子操作：**

| 子操作 | 路由 | 说明 | 参数 |
|--------|------|------|------|
| `list` | gh CLI | 列出 release | [--limit N] |
| `create` | gh CLI | 创建 release | <tag> [--title] [--notes] [--draft] [--prerelease] |
| `view` | gh CLI | 查看详情 | <tag> |
| `delete` | gh CLI | 删除 release | <tag> [--yes] |

**默认行为：** 无参数时列出最近的 release。

### 5. /git:repo

**Frontmatter:**
```yaml
description: "Repository operations: create, search, clone, fork, delete"
argument-hint: "[create|search|clone|fork|delete] [args]"
allowed-tools: ["Bash", "Read"]
```

**子操作：**

| 子操作 | 路由 | 说明 | 参数 |
|--------|------|------|------|
| `create` | MCP | 创建仓库 | <name> [--private] [--description] |
| `search` | MCP | 搜索仓库 | <query> [--language] [--sort stars/updated] |
| `clone` | gh CLI | 克隆仓库 | <owner/repo> [--depth] |
| `fork` | MCP | Fork 仓库 | <owner/repo> [--org] |
| `delete` | gh CLI | 删除仓库 | <owner/repo> [--yes] |

**默认行为：** 无参数时显示当前仓库信息。

### 6. /git:branch

**Frontmatter:**
```yaml
description: "Branch operations: list, create, delete, switch, clean"
argument-hint: "[list|create|delete|switch|clean] [args]"
allowed-tools: ["Bash(git branch:*), Bash(git checkout:*), Bash(git worktree:*)"]
```

**子操作：**

| 子操作 | 路由 | 说明 | 参数 |
|--------|------|------|------|
| `list` | MCP → git | 列出分支 | [--all] [--merged] |
| `create` | git | 创建分支 | <name> [--from] |
| `delete` | git | 删除分支 | <name> [--remote] |
| `switch` | git | 切换分支 | <name> |
| `clean` | git | 清理 gone 分支 | [--dry-run] |

**默认行为：** 无参数时列出本地分支。

### 7. /git:commit

**Frontmatter:**
```yaml
description: "Commit operations: add, commit, status, diff"
argument-hint: "[add|commit|status|diff] [args]"
allowed-tools: ["Bash(git add:*), Bash(git commit:*), Bash(git status:*), Bash(git diff:*)"]
```

**子操作：**

| 子操作 | 路由 | 说明 | 参数 |
|--------|------|------|------|
| `add` | git | 暂存文件 | [files...] [--all] [--patch] |
| `commit` | git | 创建提交 | [--message] [--amend] |
| `status` | git | 查看状态 | [--short] |
| `diff` | git | 查看变更 | [--staged] [--stat] |

**默认行为：** 无参数时执行 `git status` + `git diff HEAD`，然后自动暂存+提交（复用 commit-commands 的 commit 逻辑）。

### 8. /git:sync

**Frontmatter:**
```yaml
description: "Sync operations: pull, push, fetch, rebase"
argument-hint: "[pull|push|fetch|rebase] [args]"
allowed-tools: ["Bash(git pull:*), Bash(git push:*), Bash(git fetch:*), Bash(git rebase:*)"]
```

**子操作：**

| 子操作 | 路由 | 说明 | 参数 |
|--------|------|------|------|
| `pull` | git | 拉取更新 | [--rebase] [--autostash] |
| `push` | git | 推送 | [--force] [--set-upstream] |
| `fetch` | git | 获取远程引用 | [--all] [--prune] |
| `rebase` | git | 变基 | <target-branch> [--interactive] |

**默认行为：** 无参数时执行 `git pull --rebase`。

## 自动触发技能：git-router

**Frontmatter:**
```yaml
name: git-router
description: "Use when user mentions git, GitHub, PR, issue, branch, commit, release, CI/CD, workflow, merge, clone, fork, or any version control operation. Guides to the right /git:* command."
```

**行为：**
1. 识别用户的 git/github 相关意图
2. 通过 AskUserQuestion 推荐最合适的 `/git:*` 命令
3. 提供命令示例
4. 不执行具体操作，只做引导

## 与现有插件的关系

### 替代 commit-commands

`git-manager` 完全覆盖 `commit-commands` 的功能：

| commit-commands | git-manager |
|----------------|-------------|
| `/commit-commands:commit` | `/git:commit`（默认行为） |
| `/commit-commands:commit-push-pr` | `/git:commit` + `/git:sync push` + `/git:pr create` |
| `/commit-commands:clean_gone` | `/git:branch clean` |

**迁移步骤：**
1. 创建 `git-manager` 插件
2. 测试所有功能
3. 禁用或卸载 `commit-commands`

### 与 github MCP 插件的关系

`git-manager` 内置 MCP 配置，不需要单独的 `github` MCP 插件。

**迁移步骤：**
1. 将 `.mcp.json` 中的 GitHub 配置移到 `git-manager`
2. 禁用或卸载独立的 `github` MCP 插件

## 实现计划

### Phase 1: 基础框架
- 创建 plugin.json
- 配置 .mcp.json
- 创建 references/routing-rules.md

### Phase 2: 核心命令
- /git:commit（复用 commit-commands 逻辑）
- /git:branch
- /git:sync
- /git:pr

### Phase 3: 扩展命令
- /git:issue
- /git:ci
- /git:release
- /git:repo

### Phase 4: 自动触发
- git-router SKILL.md
- smart-router agent

### Phase 5: 测试与迁移
- 测试所有命令
- 测试路由逻辑
- 迁移 commit-commands 用户

## 成功标准

1. 所有 8 个命令可用且路由正确
2. MCP 不可用时自动降级到 gh CLI
3. gh CLI 不可用时自动降级到 git 原生
4. 自动触发技能能正确识别 git/github 意图
5. 完全覆盖 commit-commands 的所有功能
