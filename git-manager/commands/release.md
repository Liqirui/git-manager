---
description: "Release management: list, create, view, delete"
argument-hint: "[list|create|view|delete] [tag] [args]"
allowed-tools: Bash(gh release:*), Bash(gh api:*), Read
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
