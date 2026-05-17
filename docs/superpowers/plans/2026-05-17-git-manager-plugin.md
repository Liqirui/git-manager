# git-manager 插件实现计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 创建一个 Claude Code 插件，整合 GitHub MCP 和 gh CLI，提供 8 个领域分组的 git 管理命令和自动触发路由技能。

**Architecture:** 插件采用 commands/ + skills/ + agents/ + references/ 标准结构。8 个斜杠命令覆盖 git 全部操作，智能路由在每个命令内部决定用 MCP 还是 gh CLI，一个自动触发技能在用户提到 git/github 意图时引导到正确命令。

**Tech Stack:** Claude Code Plugin SDK (plugin.json + .mcp.json), GitHub MCP Server (HTTP), GitHub CLI (gh), Git (bash)

---

## 文件结构

```
git-manager/
├── .claude-plugin/
│   └── plugin.json                    # 插件元数据
├── .mcp.json                          # GitHub MCP 服务器配置
├── references/
│   └── routing-rules.md               # MCP vs gh CLI 路由规则表
├── commands/
│   ├── commit.md                      # /git:commit [add|commit|status|diff]
│   ├── branch.md                      # /git:branch [list|create|delete|switch|clean]
│   ├── sync.md                        # /git:sync [pull|push|fetch|rebase]
│   ├── pr.md                          # /git:pr [create|list|view|merge|review|diff|comment]
│   ├── issue.md                       # /git:issue [list|create|view|close|comment|label]
│   ├── ci.md                          # /git:ci [status|rerun|logs|watch]
│   ├── release.md                     # /git:release [list|create|view|delete]
│   └── repo.md                        # /git:repo [create|search|clone|fork|delete]
├── skills/
│   └── git-router/
│       └── SKILL.md                   # 自动触发路由技能
└── agents/
    └── smart-router.md                # 路由决策代理定义
```

---

## Task 1: 插件基础框架

**Files:**
- Create: `git-manager/.claude-plugin/plugin.json`
- Create: `git-manager/.mcp.json`

- [ ] **Step 1: 创建插件目录**

```bash
mkdir -p "D:/AI/claude-rest/git-manager/.claude-plugin"
```

- [ ] **Step 2: 创建 plugin.json**

```json
{
  "name": "git-manager",
  "description": "Unified Git & GitHub management with smart routing between GitHub MCP and gh CLI. Covers local git operations, PR/issue management, CI/CD, releases, and repository management.",
  "author": {
    "name": "User"
  }
}
```

- [ ] **Step 3: 创建 .mcp.json**

从现有的 `C:/Users/key/.claude/.mcp.json` 中提取 GitHub MCP 配置：

```json
{
  "mcpServers": {
    "github": {
      "command": "uvx",
      "args": ["--from", "github-mcp-server", "github-mcp-cli"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_PERSONAL_ACCESS_TOKEN}"
      }
    }
  }
}
```

- [ ] **Step 4: 验证插件结构**

```bash
ls -la "D:/AI/claude-rest/git-manager/.claude-plugin/plugin.json"
ls -la "D:/AI/claude-rest/git-manager/.mcp.json"
```

Expected: 两个文件都存在且内容正确。

- [ ] **Step 5: 提交**

```bash
cd D:/AI/claude-rest
git add git-manager/.claude-plugin/plugin.json git-manager/.mcp.json
git commit -m "feat(git-manager): initialize plugin with MCP config"
```

---

## Task 2: 路由规则参考文档

**Files:**
- Create: `git-manager/references/routing-rules.md`

- [ ] **Step 1: 创建路由规则文档**

```markdown
# Git Manager 路由规则

## 路由优先级

每个操作按以下顺序尝试：
1. 首选工具
2. 降级工具（首选不可用时自动切换）
3. 中止并提示用户配置

## 路由矩阵

### PR 操作
| 子操作 | 首选 | 降级 | 原因 |
|--------|------|------|------|
| create | MCP `create_pull_request` | `gh pr create` | MCP 结构化返回 |
| list | MCP `list_pull_requests` | `gh pr list --json` | MCP 支持分页 |
| view | MCP `pull_request_read` | `gh pr view` | MCP 返回完整数据 |
| merge | MCP `merge_pull_request` | `gh pr merge` | MCP 支持三种合并方式 |
| review | MCP `pull_request_review_write` | `gh pr review` | MCP 结构化提交 |
| diff | MCP `pull_request_read(get_diff)` | `gh pr diff` | 均可 |
| comment | MCP `add_issue_comment` | `gh pr comment` | MCP 统一 issue/PR 评论 |

### Issue 操作
| 子操作 | 首选 | 降级 | 原因 |
|--------|------|------|------|
| list | MCP `list_issues` | `gh issue list` | MCP 支持过滤 |
| create | MCP `issue_write(create)` | `gh issue create` | MCP 结构化 |
| view | MCP `issue_read(get)` | `gh issue view` | MCP 返回评论 |
| close | MCP `issue_write(update)` | `gh issue close` | MCP 支持 state_reason |
| comment | MCP `add_issue_comment` | `gh issue comment` | 均可 |
| label | MCP `issue_write` | `gh issue edit --add-label` | 均可 |

### CI/CD 操作
| 子操作 | 首选 | 降级 | 原因 |
|--------|------|------|------|
| status | `gh run list` | 无 | MCP 不支持 |
| rerun | `gh run rerun` | 无 | MCP 不支持 |
| logs | `gh run view --log` | 无 | MCP 不支持 |
| watch | `gh run watch` | 无 | MCP 不支持 |

### Release 操作
| 子操作 | 首选 | 降级 | 原因 |
|--------|------|------|------|
| list | `gh release list` | MCP `list_releases` | gh 更完整 |
| create | `gh release create` | MCP `create_release` | gh 支持 assets |
| view | `gh release view` | MCP `get_release_by_tag` | 均可 |
| delete | `gh release delete` | 无 | MCP 不支持删除 |

### Repo 操作
| 子操作 | 首选 | 降级 | 原因 |
|--------|------|------|------|
| create | MCP `create_repository` | `gh repo create` | MCP 结构化 |
| search | MCP `search_repositories` | `gh search repos` | MCP 结构化 |
| clone | `gh repo clone` | `git clone` | gh 处理认证 |
| fork | MCP `fork_repository` | `gh repo fork` | MCP 结构化 |
| delete | `gh api repos/{owner}/{repo} -X DELETE` | 无 | MCP 不支持 |

### Branch 操作
| 子操作 | 首选 | 降级 | 原因 |
|--------|------|------|------|
| list | MCP `list_branches` | `git branch -a` | MCP 含远程信息 |
| create | `git checkout -b` | MCP `create_branch` | 本地优先 |
| delete | `git branch -D` | MCP (需额外调用) | 本地优先 |
| switch | `git checkout` | 无 | 纯本地 |
| clean | `git branch -v` + `git branch -D` | 无 | 纯本地 |

### Commit 操作
| 子操作 | 首选 | 降级 | 原因 |
|--------|------|------|------|
| add | `git add` | 无 | 纯本地 |
| commit | `git commit` | 无 | 纯本地 |
| status | `git status` | 无 | 纯本地 |
| diff | `git diff` | 无 | 纯本地 |

### Sync 操作
| 子操作 | 首选 | 降级 | 原因 |
|--------|------|------|------|
| pull | `git pull` | 无 | 纯本地 |
| push | `git push` | 无 | 纯本地 |
| fetch | `git fetch` | 无 | 纯本地 |
| rebase | `git rebase` | 无 | 纯本地 |

## 降级触发条件

- MCP 工具调用返回错误（连接超时、认证失败、服务不可用）
- MCP 工具不在可用工具列表中
- gh CLI 未安装或未认证（`gh auth status` 返回非零）

## 降级行为

- 静默降级，不中断操作
- 在输出中添加一行提示："[tool] 不可用，已使用 [fallback] 完成"
- 写操作降级后验证结果一致性
```

- [ ] **Step 2: 验证文件内容**

```bash
cat "D:/AI/claude-rest/git-manager/references/routing-rules.md" | wc -l
```

Expected: 约 80-100 行。

- [ ] **Step 3: 提交**

```bash
cd D:/AI/claude-rest
git add git-manager/references/routing-rules.md
git commit -m "docs(git-manager): add routing rules reference"
```

---

## Task 3: /git:commit 命令

**Files:**
- Create: `git-manager/commands/commit.md`

此命令复用 `commit-commands` 插件的 commit 逻辑，扩展为支持子操作。

- [ ] **Step 1: 创建 commit.md**

```markdown
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
```

- [ ] **Step 2: 验证 frontmatter 格式**

```bash
head -5 "D:/AI/claude-rest/git-manager/commands/commit.md"
```

Expected: 包含 `---` 分隔的 YAML frontmatter。

- [ ] **Step 3: 提交**

```bash
cd D:/AI/claude-rest
git add git-manager/commands/commit.md
git commit -m "feat(git-manager): add /git:commit command"
```

---

## Task 4: /git:branch 命令

**Files:**
- Create: `git-manager/commands/branch.md`

- [ ] **Step 1: 创建 branch.md**

```markdown
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
2. Check if branch has associated worktree: `git worktree list`
3. If worktree exists: `git worktree remove --force <worktree-path>`
4. Delete local branch: `git branch -D <name>`
5. If `--remote` in `$ARGUMENTS`: `git push origin --delete <name>`

## Sub-command: switch

1. Extract branch name from `$ARGUMENTS`
2. Run `git checkout <name>`
3. Report current branch

## Sub-command: clean

1. Run `git branch -v` to find [gone] branches
2. For each [gone] branch:
   - Check worktree: `git worktree list`
   - Remove worktree if exists: `git worktree remove --force <path>`
   - Delete branch: `git branch -D <name>`
3. If `--dry-run` in `$ARGUMENTS`: only list, don't delete
4. Report cleaned branches
```

- [ ] **Step 2: 提交**

```bash
cd D:/AI/claude-rest
git add git-manager/commands/branch.md
git commit -m "feat(git-manager): add /git:branch command"
```

---

## Task 5: /git:sync 命令

**Files:**
- Create: `git-manager/commands/sync.md`

- [ ] **Step 1: 创建 sync.md**

```markdown
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
3. Execute and report result

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
```

- [ ] **Step 2: 提交**

```bash
cd D:/AI/claude-rest
git add git-manager/commands/sync.md
git commit -m "feat(git-manager): add /git:sync command"
```

---

## Task 6: /git:pr 命令

**Files:**
- Create: `git-manager/commands/pr.md`

此命令优先使用 MCP 工具，降级到 gh CLI。

- [ ] **Step 1: 创建 pr.md**

```markdown
---
description: "Pull request operations: create, list, view, merge, review, diff, comment"
argument-hint: "[create|list|view|merge|review|diff|comment] [number] [args]"
allowed-tools: Bash, Glob, Grep, Read
---

# /git:pr

## Context

- Current branch: !`git branch --show-current`
- Remote info: !`git remote -v`
- Recent PR-related branches: !`git branch -a | grep -i pr || true`

## Tool Routing

**Preferred:** GitHub MCP tools (mcp__plugin_github_github__*)
**Fallback:** `gh` CLI commands

For each sub-command, try MCP first. If MCP tool is not available or returns error, fall back to gh CLI.

## Sub-command Routing

Parse `$ARGUMENTS`:

- **No arguments**: List open PRs for current branch
- **create**: Create PR from current branch
- **list**: List PRs with optional filters
- **view \<number\>**: View PR details
- **merge \<number\>**: Merge PR
- **review \<number\>**: Submit review
- **diff \<number\>**: Show PR diff
- **comment \<number\>**: Add comment

## Sub-command: create

### MCP path:
1. Get current branch and repo info
2. Call `create_pull_request` with:
   - owner/repo from git remote
   - head: current branch
   - base: default branch (usually main/master)
   - title: from `$ARGUMENTS` or auto-generate from commits
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

### MCP path:
1. Call `pull_request_review_write` with:
   - event: from `--event` (default: COMMENT)
   - body: from `--body` or auto-generated

### gh fallback:
1. `gh pr review <number> --<event> --body "<body>"`

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
```

- [ ] **Step 2: 提交**

```bash
cd D:/AI/claude-rest
git add git-manager/commands/pr.md
git commit -m "feat(git-manager): add /git:pr command with MCP routing"
```

---

## Task 7: /git:issue 命令

**Files:**
- Create: `git-manager/commands/issue.md`

- [ ] **Step 1: 创建 issue.md**

```markdown
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
```

- [ ] **Step 2: 提交**

```bash
cd D:/AI/claude-rest
git add git-manager/commands/issue.md
git commit -m "feat(git-manager): add /git:issue command with MCP routing"
```

---

## Task 8: /git:ci 命令

**Files:**
- Create: `git-manager/commands/ci.md`

此命令全部使用 gh CLI（MCP 不支持 CI/CD 操作）。

- [ ] **Step 1: 创建 ci.md**

```markdown
---
description: "CI/CD operations: status, rerun, logs, watch"
argument-hint: "[status|rerun|logs|watch] [run-id]"
allowed-tools: Bash(gh run:*), Bash(gh workflow:*)
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
```

- [ ] **Step 2: 提交**

```bash
cd D:/AI/claude-rest
git add git-manager/commands/ci.md
git commit -m "feat(git-manager): add /git:ci command"
```

---

## Task 9: /git:release 命令

**Files:**
- Create: `git-manager/commands/release.md`

- [ ] **Step 1: 创建 release.md**

```markdown
---
description: "Release management: list, create, view, delete"
argument-hint: "[list|create|view|delete] [tag] [args]"
allowed-tools: Bash(gh release:*), Bash(gh api:*)
---

# /git:release

## Tool Routing

**Preferred:** `gh` CLI (more complete release support)
**Fallback:** MCP `list_releases` / `get_release_by_tag`

## Sub-command Routing

Parse `$ARGUMENTS`:

- **No arguments**: List recent releases
- **list**: List releases
- **create \<tag\>**: Create new release
- **view \<tag\>**: View release details
- **delete \<tag\>**: Delete release

## Sub-command: list (default)

1. Run: `gh release list --limit 20`
2. Display formatted table

## Sub-command: create

1. Extract tag from `$ARGUMENTS`
2. Build command: `gh release create <tag>`
3. If `--title`: add `--title "<title>"`
4. If `--notes`: add `--notes "<notes>"`
5. If `--draft`: add `--draft`
6. If `--prerelease`: add `--prerelease`
7. Execute and report

## Sub-command: view

1. Extract tag from `$ARGUMENTS`
2. Run: `gh release view <tag>`
3. Display details

## Sub-command: delete

1. Extract tag from `$ARGUMENTS`
2. Confirm with user (destructive operation)
3. Run: `gh release delete <tag> --yes`
4. Report success
```

- [ ] **Step 2: 提交**

```bash
cd D:/AI/claude-rest
git add git-manager/commands/release.md
git commit -m "feat(git-manager): add /git:release command"
```

---

## Task 10: /git:repo 命令

**Files:**
- Create: `git-manager/commands/repo.md`

- [ ] **Step 1: 创建 repo.md**

```markdown
---
description: "Repository operations: create, search, clone, fork, delete"
argument-hint: "[create|search|clone|fork|delete] [name] [args]"
allowed-tools: Bash, Read
---

# /git:repo

## Tool Routing

| 子操作 | 首选 | 降级 |
|--------|------|------|
| create | MCP | gh CLI |
| search | MCP | gh CLI |
| clone | gh CLI | git |
| fork | MCP | gh CLI |
| delete | gh CLI | 无 |

## Sub-command Routing

Parse `$ARGUMENTS`:

- **No arguments**: Show current repo info
- **create \<name\>**: Create new repository
- **search \<query\>**: Search repositories
- **clone \<owner/repo\>**: Clone repository
- **fork \<owner/repo\>**: Fork repository
- **delete \<owner/repo\>**: Delete repository

## Sub-command: info (default, no arguments)

1. Run: `gh repo view`
2. Display repo details

## Sub-command: create

### MCP path:
1. Call `create_repository` with:
   - name: from `$ARGUMENTS`
   - private: true if `--private`, false otherwise
   - description: from `--description`
   - autoInit: true

### gh fallback:
1. `gh repo create <name> --<visibility> --description "<desc>"`

## Sub-command: search

### MCP path:
1. Call `search_repositories` with:
   - query: from `$ARGUMENTS`
   - sort: from `--sort` (default: best match)

### gh fallback:
1. `gh search repos <query> --sort <sort> --limit 10`

## Sub-command: clone

1. Run: `gh repo clone <owner/repo>`
2. If `--depth`: add `--depth <n>`

## Sub-command: fork

### MCP path:
1. Call `fork_repository` with owner/repo

### gh fallback:
1. `gh repo fork <owner/repo>`
2. If `--org`: add `--org <org>`

## Sub-command: delete

1. Confirm with user (destructive)
2. Run: `gh api repos/<owner/repo> -X DELETE`
3. Report success
```

- [ ] **Step 2: 提交**

```bash
cd D:/AI/claude-rest
git add git-manager/commands/repo.md
git commit -m "feat(git-manager): add /git:repo command"
```

---

## Task 11: 自动触发技能 git-router

**Files:**
- Create: `git-manager/skills/git-router/SKILL.md`

- [ ] **Step 1: 创建 SKILL.md**

```markdown
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
```

- [ ] **Step 2: 提交**

```bash
cd D:/AI/claude-rest
git add git-manager/skills/git-router/SKILL.md
git commit -m "feat(git-manager): add git-router auto-trigger skill"
```

---

## Task 12: 路由决策代理

**Files:**
- Create: `git-manager/agents/smart-router.md`

- [ ] **Step 1: 创建 smart-router.md**

```markdown
# Smart Router Agent

You are the routing decision maker for the git-manager plugin. When a command needs to decide between MCP and gh CLI, you make the decision.

## Decision Process

1. Check if MCP tools are available (look for `mcp__plugin_github_github__*` tools)
2. Check if `gh` CLI is available (run `gh auth status`)
3. Consult the routing rules in `references/routing-rules.md`
4. Execute with the preferred tool
5. If preferred tool fails, execute with fallback

## MCP Tool Availability Check

If you can call `mcp__plugin_github_github__get_me` successfully, MCP is available.

## gh CLI Availability Check

If `gh auth status` returns exit code 0, gh CLI is available.

## Routing Rules Summary

- **MCP preferred:** PR ops, Issue ops, Code search, File ops, Repo create/search/fork, Branch list
- **gh CLI preferred:** CI/CD, Release, Secrets/Variables, Clone, Delete repo, Any API
- **git native:** Commit, Branch create/delete/switch, Pull, Push, Fetch, Rebase

## Fallback Behavior

When falling back:
1. Log which tool was used: "[tool] 不可用，已使用 [fallback]"
2. Verify the operation succeeded
3. For write operations, confirm the result matches expectations
```

- [ ] **Step 2: 提交**

```bash
cd D:/AI/claude-rest
git add git-manager/agents/smart-router.md
git commit -m "feat(git-manager): add smart-router agent"
```

---

## Task 13: 全局配置更新

**Files:**
- Modify: `C:/Users/key/.claude/.mcp.json` (移除旧 GitHub MCP 配置)

- [ ] **Step 1: 读取当前配置**

```bash
cat "C:/Users/key/.claude/.mcp.json"
```

- [ ] **Step 2: 移除 github MCP 条目**

从 `.mcp.json` 的 `mcpServers` 中移除 `github` 条目（因为它现在在 git-manager 插件的 `.mcp.json` 中）。

修改后的文件应只保留 `fetch` 服务器：
```json
{
  "mcpServers": {
    "fetch": {
      "command": "uvx",
      "args": ["mcp-server-fetch"]
    }
  }
}
```

- [ ] **Step 3: 验证配置**

重启 Claude Code 后，验证：
- `mcp__plugin_github_github__*` 工具仍然可用（来自 git-manager 插件）
- `/git:*` 命令可用

---

## Task 14: 集成验证

- [ ] **Step 1: 验证插件加载**

在 Claude Code 中运行：
```
/git:commit status
```
Expected: 显示当前 git 状态。

- [ ] **Step 2: 验证 MCP 路由**

```
/git:pr list
```
Expected: 使用 MCP 工具列出 PR（或降级到 gh CLI）。

- [ ] **Step 3: 验证 gh CLI 路由**

```
/git:ci status
```
Expected: 使用 gh CLI 显示 workflow 状态。

- [ ] **Step 4: 验证自动触发**

在对话中输入："帮我创建一个 PR"
Expected: git-router 技能触发，建议使用 `/git:pr create`。

- [ ] **Step 5: 验证降级**

临时禁用 MCP（断开网络），运行：
```
/git:pr list
```
Expected: 自动降级到 gh CLI，显示提示信息。

---

## 实现顺序

1. Task 1-2: 基础框架（plugin.json, .mcp.json, routing-rules.md）
2. Task 3-5: 本地 git 命令（commit, branch, sync）
3. Task 6-7: GitHub 核心命令（pr, issue）
4. Task 8-10: GitHub 扩展命令（ci, release, repo）
5. Task 11-12: 自动触发技能和路由代理
6. Task 13-14: 配置迁移和集成验证

## 回滚计划

如果新插件有问题：
1. 恢复 `C:/Users/key/.claude/.mcp.json` 中的 github 配置
2. 重新启用 `commit-commands` 插件
3. 删除 `git-manager` 目录
