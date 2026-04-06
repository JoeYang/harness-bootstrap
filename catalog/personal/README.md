# Personal Claude Code Configuration Catalog

Joe's global `~/.claude/CLAUDE.md` documented with reasoning for every section.

## Purpose

This catalog is the seed knowledge for the harness bootstrap system. Each setting
is analyzed with:

- **What** it does and where it lives (line numbers)
- **Why** it exists (what goes wrong without it)
- **When useful** vs **when NOT useful** (project types, scenarios)
- **Candidate for**: Global (stays in `~/.claude/`), Project-level (moves to
  `.claude/`), or Both (global default + project-level override, e.g. Slack
  channel, branching details)

## How this feeds the bootstrap

Settings marked "Global only" stay in `~/.claude/CLAUDE.md` and apply everywhere.
Settings marked "Project-level only" should be injected into a project's `.claude/CLAUDE.md`
only when that project type matches. Settings marked "Both" have a global default
with project-level overrides.

See `claude-md-audit.md` for the full section-by-section analysis.
