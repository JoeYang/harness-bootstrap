# Claude Code Harness

Knowledge base and tooling for Claude Code harness configuration (`~/.claude/`, `.claude/`).

## Problem

The global `~/.claude/CLAUDE.md` is 224 lines. Every project loads all 224 lines -- exchange smoke tests pollute Python CLIs, 60 lines of agent team orchestration load in single-agent sessions, trading-specific rules appear in study projects.

## Solution

Trim the global config to ~90 lines of truly universal rules. Move project-specific rules into each project's `.claude/` directory. Build tooling to do this efficiently across 17+ projects.

```
BEFORE: ~/.claude/CLAUDE.md (224 lines -> every project)
AFTER:  ~/.claude/CLAUDE.md (~90 lines -> universal only)
        + each project's .claude/ (project-specific only)
```

## What's Inside

| Directory | Purpose |
|---|---|
| `catalog/` | Joe's existing config documented with reasoning (seed knowledge) |
| `docs/` | Harness anatomy, cookbooks, capability catalog, audit reference |
| `examples/` | Complete reference configs the skills read when generating |
| `rules-library/` | Reusable rule files copied into projects |
| `configs/` | Saved bootstrap answers for config reuse |
| `skills/` | `/harness-bootstrap` and `/harness-update` |

## When to Use What

| Tool | Purpose |
|---|---|
| `/harness-bootstrap` | Initial `.claude/` setup for a new or existing project |
| `/harness-update` | Add newly cataloged capabilities to an existing project |
| `/claude-automation-recommender` | Read-only advisory on what automations to add (upstream) |
| superpowers skills | Workflow orchestration (plans, TDD, etc.) -- we reference, not replace |

## Quick Start

1. Clone this repo and set the env var so skills can find it:
   ```bash
   export HARNESS_DIR="$HOME/clawd/harness"
   ```

2. Symlink the skills into your global Claude config:
   ```bash
   ln -s "$HARNESS_DIR/skills/harness-bootstrap" ~/.claude/skills/harness-bootstrap
   ln -s "$HARNESS_DIR/skills/harness-update" ~/.claude/skills/harness-update
   ```

3. Then use in any project:
   ```
   /harness-bootstrap   # first-time setup
   /harness-update      # add new capabilities later
   ```

## See Also

- `docs/global-audit.md` -- what stays global vs moves to project level
- `docs/plan-v5.md` -- full implementation plan
