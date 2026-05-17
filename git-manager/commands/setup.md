---
description: "Verify environment dependencies and plugin installation"
argument-hint: ""
allowed-tools: Bash, Read
---

# /git:setup

Verify that all dependencies for the git-manager plugin are properly installed and configured.

## Checks

Run each check and report results:

### 1. Git

```bash
git --version
```

Expected: `git version 2.x.x` or higher.
If missing: Install from https://git-scm.com/

### 2. GitHub CLI

```bash
gh --version 2>/dev/null
```

Expected: `gh version 2.x.x` or higher.
If missing: Install via `winget install GitHub.cli` or https://cli.github.com/

### 3. GitHub CLI Authentication

```bash
gh auth status 2>&1
```

Expected: Shows logged-in user.
If not authenticated: Run `gh auth login`

### 4. uv/uvx (MCP Server dependency)

```bash
uvx --version 2>/dev/null || uv --version 2>/dev/null
```

Expected: `uv 0.4.x` or higher.
If missing: Install via `pip install uv` or `winget install astral-sh.uv`

### 5. GitHub Personal Access Token

Check if environment variable is set:

```bash
# PowerShell
if ($env:GITHUB_PERSONAL_ACCESS_TOKEN) { "Token: SET (length: $($env:GITHUB_PERSONAL_ACCESS_TOKEN.Length))" } else { "Token: NOT SET" }
```

Expected: Token is set with length > 0.
If not set: Create at https://github.com/settings/tokens (scopes: repo, read:org, workflow), then set `GITHUB_PERSONAL_ACCESS_TOKEN` environment variable.

### 6. MCP Server Connectivity

Attempt to verify MCP tools are available. If MCP tools like `mcp__plugin_github_github__get_me` are accessible, MCP is working.

## Output Format

Present results as a checklist:

```
## Git Manager 环境检查

- [x] git 2.43.0 ✓
- [x] gh 2.65.0 ✓
- [x] gh auth: logged in as username ✓
- [x] uvx 0.5.x ✓
- [x] GITHUB_PERSONAL_ACCESS_TOKEN: set ✓
- [x] MCP Server: connected ✓

All checks passed! Plugin is ready to use.
```

If any check fails, show:

```
- [ ] gh CLI: NOT INSTALLED
      Install: winget install GitHub.cli
```

## Quick Fix Commands

If something is missing, provide the exact fix command:

| Missing | Fix (Windows) | Fix (macOS) |
|---------|--------------|-------------|
| git | `winget install Git.Git` | `brew install git` |
| gh | `winget install GitHub.cli` | `brew install gh` |
| uv/uvx | `pip install uv` | `brew install uv` |
| Token | Set `$env:GITHUB_PERSONAL_ACCESS_TOKEN` | Set `export GITHUB_PERSONAL_ACCESS_TOKEN` |
