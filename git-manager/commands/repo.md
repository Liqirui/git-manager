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
