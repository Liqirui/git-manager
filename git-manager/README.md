# git-manager

Claude Code 插件：统一 Git & GitHub 管理，智能路由 MCP 和 gh CLI。

## 前置依赖

| 依赖 | 最低版本 | 用途 | 安装 |
|------|---------|------|------|
| git | 2.x | 本地版本控制 | [git-scm.com](https://git-scm.com/) |
| gh CLI | 2.x | GitHub 远程操作 | `winget install GitHub.cli` 或 [cli.github.com](https://cli.github.com/) |
| uv/uvx | 0.4+ | 启动 GitHub MCP Server | `pip install uv` 或 `winget install astral-sh.uv` |

## 安装步骤

### 1. 安装依赖

```bash
# Windows (winget)
winget install GitHub.cli
winget install astral-sh.uv

# macOS (brew)
brew install gh
brew install uv

# Linux (Debian/Ubuntu)
sudo apt install gh
pip install uv
```

### 2. 认证 GitHub CLI

```bash
gh auth login
```

按提示选择 GitHub.com → HTTPS → 浏览器登录。

### 3. 设置 GitHub Token（MCP 用）

创建 Personal Access Token: https://github.com/settings/tokens

所需权限：`repo`, `read:org`, `workflow`

设置环境变量：
```bash
# Windows PowerShell
$env:GITHUB_PERSONAL_ACCESS_TOKEN = "ghp_xxxxxxxxxxxx"
# 持久化：写入 $PROFILE

# macOS/Linux
export GITHUB_PERSONAL_ACCESS_TOKEN="ghp_xxxxxxxxxxxx"
# 持久化：写入 ~/.bashrc 或 ~/.zshrc
```

### 4. 安装插件

将 `git-manager/` 目录复制到 Claude Code 插件目录：

```bash
# Windows
xcopy /E /I git-manager "%USERPROFILE%\.claude\plugins\git-manager"

# macOS/Linux
cp -r git-manager ~/.claude/plugins/git-manager
```

或在 Claude Code 中通过插件市场安装。

### 5. 验证安装

在 Claude Code 中运行：

```
/git:setup
```

自动检查所有依赖和配置是否就绪。如果全部通过，即可开始使用。

## 命令列表

| 命令 | 说明 | 示例 |
|------|------|------|
| `/git:setup` | 环境校验 | `/git:setup` |
| `/git:commit` | 提交操作 | `/git:commit` 或 `/git:commit add file.py` |
| `/git:branch` | 分支操作 | `/git:branch create feature-x` |
| `/git:sync` | 同步操作 | `/git:sync push` |
| `/git:pr` | PR 管理 | `/git:pr create` 或 `/git:pr list` |
| `/git:issue` | Issue 管理 | `/git:issue create --title "bug"` |
| `/git:ci` | CI/CD 操作 | `/git:ci status` |
| `/git:release` | Release 管理 | `/git:release create v1.0.0` |
| `/git:repo` | 仓库操作 | `/git:repo create my-project` |

## 智能路由

插件自动选择最优工具：

- **PR/Issue/Repo 操作** → 优先 GitHub MCP，降级 gh CLI
- **CI/CD/Release 操作** → 优先 gh CLI
- **本地 git 操作** → git 原生命令

降级时会提示：`[工具] 不可用，已使用 [备选] 完成`
