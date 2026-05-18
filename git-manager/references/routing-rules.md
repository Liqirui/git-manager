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
| review | `gh pr review` | 无 | MCP 无 review 写入工具 |
| diff | MCP `pull_request_read(get_diff)` | `gh pr diff` | 均可 |
| comment | MCP `add_issue_comment` | `gh pr comment` | MCP 统一 issue/PR 评论 |

### Issue 操作
| 子操作 | 首选 | 降级 | 原因 |
|--------|------|------|------|
| list | MCP `list_issues` | `gh issue list` | MCP 支持过滤 |
| create | `gh issue create` | 无 | MCP 无 issue 写入工具 |
| view | MCP `issue_read(get)` | `gh issue view` | MCP 返回评论 |
| close | `gh issue close` | 无 | MCP 无 issue 写入工具 |
| comment | MCP `add_issue_comment` | `gh issue comment` | 均可 |
| label | `gh issue edit --add-label` | 无 | MCP 无 issue 写入工具 |

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
| create | `gh release create` | 无 | MCP 无 release 创建工具 |
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
