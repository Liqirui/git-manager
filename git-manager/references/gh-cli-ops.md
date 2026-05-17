# GitHub CLI 操作速查

## PR 操作
- `gh pr create` — 创建 PR
- `gh pr list` — 列出 PR
- `gh pr view <number>` — 查看 PR
- `gh pr merge <number>` — 合并 PR（--squash/--rebase/--merge）
- `gh pr review <number>` — 审查 PR（--approve/--request-changes/--comment）
- `gh pr diff <number>` — 查看 diff
- `gh pr comment <number>` — 添加评论
- `gh pr close <number>` — 关闭 PR
- `gh pr checks <number>` — 查看检查状态

## Issue 操作
- `gh issue create` — 创建 issue
- `gh issue list` — 列出 issue
- `gh issue view <number>` — 查看 issue
- `gh issue close <number>` — 关闭 issue
- `gh issue comment <number>` — 添加评论
- `gh issue edit <number>` — 编辑 issue（--add-label/--remove-label/--add-assignee）

## CI/CD 操作
- `gh run list` — 列出 workflow 运行
- `gh run view <id>` — 查看运行详情
- `gh run view <id> --log` — 查看日志
- `gh run rerun <id>` — 重跑（--failed-only）
- `gh run watch <id>` — 实时监控
- `gh run cancel <id>` — 取消运行
- `gh workflow list` — 列出 workflow
- `gh workflow run <name>` — 触发 workflow
- `gh workflow view <name>` — 查看 workflow

## Release 操作
- `gh release create <tag>` — 创建 release（--title/--notes/--draft/--prerelease）
- `gh release list` — 列出 release
- `gh release view <tag>` — 查看 release
- `gh release delete <tag>` — 删除 release
- `gh release download <tag>` — 下载 assets
- `gh release upload <tag> <files>` — 上传 assets

## 仓库操作
- `gh repo create <name>` — 创建仓库
- `gh repo clone <owner/repo>` — 克隆仓库
- `gh repo fork <owner/repo>` — Fork 仓库
- `gh repo view` — 查看当前仓库
- `gh repo delete <owner/repo>` — 删除仓库

## 其他
- `gh api <endpoint>` — 任意 GitHub API 调用
- `gh auth status` — 认证状态
- `gh secret list` — 列出 secrets
- `gh secret set <name>` — 设置 secret
- `gh variable list` — 列出 variables
- `gh variable set <name>` — 设置 variable
- `gh gist create` — 创建 gist
- `gh gist list` — 列出 gist
- `gh search repos <query>` — 搜索仓库
- `gh search code <query>` — 搜索代码
