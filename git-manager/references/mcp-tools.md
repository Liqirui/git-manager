# GitHub MCP 工具速查

> 工具名前缀 `mcp__plugin_github_github__` 省略，以短名列出。
> ⚠ 标记的工具在当前 MCP 版本中**不存在**，相关操作需降级到 gh CLI。

## PR 操作
- `create_pull_request` — 创建 PR
- `list_pull_requests` — 列出 PR
- `pull_request_read` — 查看 PR（get/get_diff/get_files/get_reviews/get_comments/get_check_runs）
- `merge_pull_request` — 合并 PR（merge/squash/rebase）
- `update_pull_request` — 更新 PR
- `update_pull_request_branch` — 更新 PR 分支
- `add_comment_to_pending_review` — 给待提交审查添加评论
- `add_reply_to_pull_request_comment` — 回复 PR 评论
- `request_copilot_review` — 请求 Copilot 审查

## Issue 操作
- `list_issues` — 列出 issue
- `issue_read` — 查看 issue（get/get_comments/get_sub_issues/get_labels）
- `add_issue_comment` — 添加评论
- `sub_issue_write` — 管理子 issue
- `list_issue_types` — 列出 issue 类型
- `get_label` — 获取标签详情

## 仓库操作
- `search_repositories` — 搜索仓库
- `create_repository` — 创建仓库
- `fork_repository` — Fork 仓库
- `get_file_contents` — 读取文件
- `create_or_update_file` — 创建/更新文件
- `delete_file` — 删除文件
- `push_files` — 批量推送文件

## 分支/标签
- `list_branches` — 列出分支
- `create_branch` — 创建分支
- `list_tags` — 列出标签
- `get_tag` — 查看标签

## Release
- `list_releases` — 列出 release
- `get_latest_release` — 最新 release
- `get_release_by_tag` — 按 tag 查看

## 搜索
- `search_code` — 搜索代码
- `search_issues` — 搜索 issue
- `search_pull_requests` — 搜索 PR
- `search_users` — 搜索用户

## 团队
- `get_me` — 当前用户信息
- `get_teams` — 获取团队列表
- `get_team_members` — 获取团队成员

## Commit
- `get_commit` — 查看 commit
- `list_commits` — 列出 commit

## 安全
- `run_secret_scanning` — 密钥扫描
