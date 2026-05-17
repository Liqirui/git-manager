# GitHub MCP 工具速查

## PR 操作
- `mcp__plugin_github_github__create_pull_request` — 创建 PR
- `mcp__plugin_github_github__list_pull_requests` — 列出 PR
- `mcp__plugin_github_github__pull_request_read` — 查看 PR（get/get_diff/get_files/get_reviews/get_comments/get_check_runs）
- `mcp__plugin_github_github__merge_pull_request` — 合并 PR（merge/squash/rebase）
- `mcp__plugin_github_github__pull_request_review_write` — 提交审查（create/submit_pending/delete_pending/resolve_thread/unresolve_thread）
- `mcp__plugin_github_github__update_pull_request` — 更新 PR
- `mcp__plugin_github_github__update_pull_request_branch` — 更新 PR 分支

## Issue 操作
- `mcp__plugin_github_github__list_issues` — 列出 issue
- `mcp__plugin_github_github__issue_read` — 查看 issue（get/get_comments/get_sub_issues/get_labels）
- `mcp__plugin_github_github__issue_write` — 创建/更新 issue
- `mcp__plugin_github_github__add_issue_comment` — 添加评论
- `mcp__plugin_github_github__sub_issue_write` — 管理子 issue

## 仓库操作
- `mcp__plugin_github_github__search_repositories` — 搜索仓库
- `mcp__plugin_github_github__create_repository` — 创建仓库
- `mcp__plugin_github_github__fork_repository` — Fork 仓库
- `mcp__plugin_github_github__get_file_contents` — 读取文件
- `mcp__plugin_github_github__create_or_update_file` — 创建/更新文件
- `mcp__plugin_github_github__delete_file` — 删除文件
- `mcp__plugin_github_github__push_files` — 批量推送文件

## 分支/标签
- `mcp__plugin_github_github__list_branches` — 列出分支
- `mcp__plugin_github_github__create_branch` — 创建分支
- `mcp__plugin_github_github__list_tags` — 列出标签
- `mcp__plugin_github_github__get_tag` — 查看标签

## Release
- `mcp__plugin_github_github__list_releases` — 列出 release
- `mcp__plugin_github_github__get_latest_release` — 最新 release
- `mcp__plugin_github_github__get_release_by_tag` — 按 tag 查看

## 搜索
- `mcp__plugin_github_github__search_code` — 搜索代码
- `mcp__plugin_github_github__search_issues` — 搜索 issue
- `mcp__plugin_github_github__search_pull_requests` — 搜索 PR
- `mcp__plugin_github_github__search_users` — 搜索用户

## 其他
- `mcp__plugin_github_github__get_me` — 当前用户信息
- `mcp__plugin_github_github__get_commit` — 查看 commit
- `mcp__plugin_github_github__list_commits` — 列出 commit
- `mcp__plugin_github_github__run_secret_scanning` — 密钥扫描
- `mcp__plugin_github_github__request_copilot_review` — 请求 Copilot 审查
