# Harness Anatomy

Reference card for every file in a Claude Code harness (`.claude/` and `~/.claude/`).

---

## CLAUDE.md Scopes and Precedence

Files concatenate -- they do NOT override each other. The harness walks up the directory tree, finds all matching files, and concatenates them into a single prompt. Later-loaded files appear last in context (highest effective precedence since LLMs weight recent context more heavily).

| Scope | Path | Committed | Loading order |
|---|---|---|---|
| Managed (org) | `/etc/claude-code/CLAUDE.md` (Linux) | N/A | First (lowest) |
| User (global) | `~/.claude/CLAUDE.md` | No | Second |
| Project | `./CLAUDE.md` or `./.claude/CLAUDE.md` | Yes | Third |
| Local | `./CLAUDE.local.md` | No (gitignored) | Last (highest) |

**Directory walk**: If you open Claude Code in `src/api/`, the harness checks for CLAUDE.md at every level from repo root to cwd. All found files are concatenated.

---

## settings.json Scopes and Precedence

Unlike CLAUDE.md, settings use key-level merging with strict precedence (highest wins).

| Scope | Path | Committed | Precedence |
|---|---|---|---|
| Managed (org) | Org-deployed config | N/A | 1 (highest) |
| Local | `.claude/settings.local.json` | No (gitignored) | 2 |
| Project | `.claude/settings.json` | Yes | 3 |
| User (global) | `~/.claude/settings.json` | No | 4 (lowest) |

**Merging**: `permissions.deny` always wins across all scopes -- if any scope denies an action, it is denied regardless of what other scopes allow.

---

## @ Reference Syntax

Pull external content into CLAUDE.md without inlining it:

```markdown
@docs/architecture.md
@~/.claude/rules.md
@../shared/conventions.md
```

The referenced file's content is loaded at the position of the `@` reference. Paths are relative to the CLAUDE.md file containing the reference, or absolute from `~`.

---

## rules/ Directory

`.claude/rules/*.md` -- path-scoped rules that load conditionally.

```yaml
---
paths:
  - "src/api/**/*.ts"
---
All API handlers must validate request bodies with zod schemas.
```

Rules load when Claude accesses (reads, edits, or analyzes) a file matching the `paths` glob. Without frontmatter, the rule loads unconditionally (same as putting it in CLAUDE.md).

---

## skills/ Directory

`.claude/skills/{name}/SKILL.md` -- on-demand knowledge loaded by trigger phrase or `/slash-command`.

Skills are NOT loaded into context by default. They activate when invoked via `/slash-command`, when a matching trigger fires, or when an agent explicitly calls the skill. Use skills for specialized workflows that would waste context if always loaded.

---

## agents/ Directory

`.claude/agents/{name}.md` -- specialized sub-agents with optional model selection and tool restrictions. Invoked explicitly (e.g., `/agent:reviewer`) and run in their own context.

---

## MCP Servers

| File | Scope | Committed | Purpose |
|---|---|---|---|
| `.mcp.json` | Project | Yes | Project-specific MCP servers (shared with team) |
| `~/.claude/settings.json` `mcpServers` | Global | No | Personal MCP servers (all projects) |

Project `.mcp.json` servers merge with global servers. If both define the same server name, the project definition is expected to take precedence.

---

## Key Files Summary

| File | Scope | Committed | Purpose |
|---|---|---|---|
| `~/.claude/CLAUDE.md` | Global | No | Personal coding standards, universal rules |
| `./CLAUDE.md` | Project | Yes | Project commands, architecture, boundaries |
| `./CLAUDE.local.md` | Project | No | Personal project overrides |
| `.claude/settings.json` | Project | Yes | Team permissions, hooks, env vars |
| `.claude/settings.local.json` | Project | No | Personal permission overrides |
| `~/.claude/settings.json` | Global | No | Global permissions, MCP, plugins, flags |
| `.claude/rules/*.md` | Project | Yes | Path-scoped conditional rules |
| `.claude/skills/*/SKILL.md` | Project | Yes | On-demand specialized workflows |
| `.claude/agents/*.md` | Project | Yes | Constrained sub-agents |
| `.mcp.json` | Project | Yes | Project MCP server definitions |
