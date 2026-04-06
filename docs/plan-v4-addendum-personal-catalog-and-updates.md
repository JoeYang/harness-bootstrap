# Plan v4 Addendum: Personal Settings Catalog & Harness Updates

> **Date**: 2026-04-06
> **Supplements**: plan-v4.md + plan-v4-addendum-capability-catalog.md

## Two New Requirements

### 1. Personal Settings Cataloged with Reasoning

Joe's existing personal config (`~/.claude/CLAUDE.md`, `settings.json`, agents, hooks, skills) gets documented in the harness repo with full reasoning: what each setting does, why it exists, when it's useful, when NOT to use it.

**Why this matters:**
- The harness repo becomes the **single source of truth** for all harness knowledge
- Joe's real-world config is the **seed knowledge** — battle-tested settings with context
- New users (or future Joe) can understand the reasoning behind every choice
- Settings without clear reasoning get identified and reconsidered

### 2. Continuous Learning + Back-Propagation via `/harness-update`

When Joe discovers a new harness pattern (hook recipe, rule, capability), he:
1. Adds it to the capability catalog
2. Optionally adds a rule to `rules-library/` or an example to `examples/`
3. Runs `/harness-update` in any existing project to pick up the new knowledge

This closes the loop: **learn once → catalog once → propagate to many projects**.

---

## Personal Settings Catalog

New directory: `catalog/personal/`

```
harness/
├── catalog/
│   └── personal/
│       ├── README.md                  # Overview of personal config philosophy
│       ├── claude-md-audit.md         # Every section of ~/.claude/CLAUDE.md with reasoning
│       ├── settings-audit.md          # Every setting in ~/.claude/settings.json with reasoning
│       ├── agents-audit.md            # Each agent: purpose, model choice, when to use
│       ├── hooks-audit.md             # Each hook: what it enforces, why deterministic
│       ├── skills-audit.md            # Each skill: purpose, invocation pattern
│       └── plugins-audit.md           # Each enabled plugin: what it provides, why enabled
```

### Format for Each Entry

```markdown
## {Setting Name}

**What**: {one-line description}
**Where**: {file path and section}
**Value**: {current value or content}

**Why this exists**: {the reasoning — what problem it solves, what incident prompted it}
**When it's useful**: {project types, scenarios where this matters}
**When NOT to use it**: {projects or scenarios where this is noise or harmful}
**Candidate for**: {global / project-level / both}

---
```

### Example Entries

#### From `claude-md-audit.md`:

```markdown
## Exchange Smoke Test Gate

**What**: Requires `./smoke_test_all.sh <exchange>` to pass before any exchange implementation is considered done
**Where**: ~/.claude/CLAUDE.md, line 94-96
**Value**: "Every exchange implementation MUST pass ./smoke_test_all.sh <exchange>..."

**Why this exists**: Exchange implementations have a complex pipeline (sim startup → order entry → matching → market data → observer display). Unit tests alone don't catch integration failures between these stages. The smoke test validates the full pipeline end-to-end.
**When it's useful**: exchange-connectivity, agency-trading — any project implementing exchange protocols
**When NOT to use it**: Python CLIs, web apps, knowledge graphs, study projects — anything not touching exchange connectivity
**Candidate for**: Project-level only (exchange projects)

---

## Agent Team Orchestration

**What**: 60 lines of rules governing how lead agents coordinate with teammate agents
**Where**: ~/.claude/CLAUDE.md, lines 155-215
**Value**: Lead behaviour, teammate lifecycle, task protocol, worktree discipline, merge strategy, delegate mode, swim lane tracking

**Why this exists**: Without these rules, lead agents try to implement code directly instead of delegating. Teammates write to the main working directory instead of worktrees. Merge conflicts arise when multiple agents edit the same files. Swim lanes keep Joe informed of parallel progress.
**When it's useful**: Any project using agent teams (CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1). Currently: exchange-connectivity, market-data-feedhandler, agency-trading.
**When NOT to use it**: Single-agent sessions. Most projects. Loading 60 lines of team orchestration rules when there's no team is pure noise.
**Candidate for**: Project-level only (projects that use agent teams)

---
```

#### From `settings-audit.md`:

```markdown
## Notification Hook (notify-send + sound)

**What**: Desktop notification with sound when Claude needs attention
**Where**: ~/.claude/settings.json → hooks.Notification
**Value**: `notify-send` + `paplay message-new-instant.oga`

**Why this exists**: Long-running agent tasks can take minutes. Without notification, Joe has to watch the terminal. The sound alert means he can work on other things and be notified when input is needed.
**When it's useful**: Always — this is a personal workflow preference
**When NOT to use it**: Headless/CI environments, or if using a different notification system
**Candidate for**: Global only (personal preference, not project-specific)

---

## sim-exchange MCP Server

**What**: Local gRPC exchange simulator at localhost:50051
**Where**: ~/.claude/settings.json → mcpServers.sim-exchange
**Value**: stdio Python process at /home/joeyang/Claude-code/sim-exchange/mcp/server.py

**Why this exists**: Provides a simulated exchange environment for testing order entry, matching, and market data without connecting to real exchanges. Used during exchange-connectivity development.
**When it's useful**: exchange-connectivity, agency-trading — when developing against the simulator
**When NOT to use it**: Any non-trading project. The MCP server won't even start if the sim isn't running.
**Candidate for**: Project-level MCP config (.mcp.json) in exchange projects only. Should NOT be in global settings — it fails silently when sim isn't running.

---
```

#### From `agents-audit.md`:

```markdown
## low-latency-trading-engineer (Opus)

**What**: Specialized agent for ultra-low-latency C++/Java trading system implementation
**Where**: ~/.claude/agents/low-latency-trading-engineer.md
**Model**: Opus (requires deep reasoning for latency analysis)

**Why this exists**: Trading system code requires specific expertise: lock-free data structures, cache-line alignment, kernel bypass networking, JNI bridges, latency measurement. A general-purpose agent makes naive choices (e.g., using std::mutex in hot paths).
**When it's useful**: market-data-feedhandler, exchange-connectivity, agency-trading — any latency-critical C++ or Java code
**When NOT to use it**: Python CLIs, web apps, study projects. The Opus model cost is unjustified for non-performance-critical code.
**Candidate for**: Global agent (available everywhere) but referenced in project CLAUDE.md only when relevant

---
```

### What the Audit Reveals

Beyond documentation, the audit identifies **action items**:

- Settings that should move from global to project-level (like exchange smoke test)
- Settings that are global but fail silently in non-applicable projects (like sim-exchange MCP)
- Settings with no clear reasoning (candidates for removal)
- Duplicate or overlapping settings
- Missing settings (gaps the capability catalog should fill)

---

## The `/harness-update` Skill

### Purpose

Incrementally update an existing project's `.claude/` config with newly cataloged capabilities.

### Flow

```
User runs /harness-update in an existing project
         │
         ▼
┌──────────────────────────────────┐
│ 1. Read project's current        │ What .claude/ files exist?
│    .claude/ config               │ What rules are installed?
│                                  │ What hooks are configured?
└────────────┬─────────────────────┘
             ▼
┌──────────────────────────────────┐
│ 2. Read capability catalog       │ Full list of available capabilities
│    ($HARNESS_DIR/docs/           │ with signal detection
│     capability-catalog.md)       │
└────────────┬─────────────────────┘
             ▼
┌──────────────────────────────────┐
│ 3. Scan project for new signals  │ Has the project added new deps,
│                                  │ frameworks, or patterns since
│                                  │ last bootstrap/update?
└────────────┬─────────────────────┘
             ▼
┌──────────────────────────────────┐
│ 4. Diff: catalog vs installed    │ What capabilities are available
│                                  │ but not yet in this project?
│                                  │ What's new since last run?
└────────────┬─────────────────────┘
             ▼
┌──────────────────────────────────┐
│ 5. Present update menu           │ "New capabilities available:
│                                  │   🆕 API design rules (FastAPI added)
│                                  │   🆕 Auto-lint hook (ruff added)
│                                  │   📦 Database rules (v2 updated)
│                                  │  Select which to add."
└────────────┬─────────────────────┘
             ▼
┌──────────────────────────────────┐
│ 6. Apply selected updates        │ Add new rules, update hooks,
│    (show diff, confirm)          │ modify settings.json
└────────────┬─────────────────────┘
             ▼
┌──────────────────────────────────┐
│ 7. Update project's config       │ Record what version of catalog
│    version marker                │ this project is synced to
└──────────────────────────────────┘
```

### Update Menu Categories

```
/harness-update

📋 Project: exchange-connectivity
   Last synced: 2026-04-06 (catalog v3)
   Current catalog: v5

🆕 New capabilities since last sync:
   [ ] gRPC protobuf rules (detected: .proto files)
   [ ] Auto-lint hook v2 (improved ruff integration)
   [ ] CI/CD awareness rules (detected: .github/workflows/)

📦 Updated capabilities:
   [ ] Testing rules v2 → v3 (added property-based testing section)
   [ ] Security rules v1 → v2 (added supply chain security)

✅ Already installed (no changes):
   • Exchange conventions (v1)
   • Agent team orchestration (v1)
   • C++ language rules (v1)
   • Auto-format hook (v1)

Select capabilities to add/update, or 'all' for everything.
```

### Version Tracking

Each project's `.claude/settings.local.json` (gitignored) stores a version marker:

```json
{
  "harness": {
    "catalog_version": "5",
    "last_synced": "2026-04-06",
    "installed_capabilities": [
      "testing-rules:v3",
      "security-rules:v2",
      "exchange:v1",
      "agent-teams:v1",
      "cpp-rules:v1",
      "auto-format-hook:v1"
    ]
  }
}
```

This lets `/harness-update` know what's already installed and what's new.

---

## Updated Repo Structure

```diff
 harness/
+├── catalog/
+│   └── personal/
+│       ├── README.md                  # Personal config philosophy
+│       ├── claude-md-audit.md         # ~/.claude/CLAUDE.md with reasoning
+│       ├── settings-audit.md          # settings.json with reasoning
+│       ├── agents-audit.md            # Agents with reasoning
+│       ├── hooks-audit.md             # Hooks with reasoning
+│       ├── skills-audit.md            # Skills with reasoning
+│       └── plugins-audit.md           # Plugins with reasoning
 ├── docs/
 │   ├── capability-catalog.md          # Signal → capability mapping (versioned)
 │   └── (rest unchanged)
 ├── examples/
 ├── rules-library/
 ├── configs/
-└── skill/
-    └── SKILL.md                        # /harness-bootstrap
+└── skills/
+    ├── harness-bootstrap/
+    │   └── SKILL.md                    # /harness-bootstrap (initial setup)
+    └── harness-update/
+        └── SKILL.md                    # /harness-update (incremental updates)
```

### Updated Implementation Plan

Add these commits to plan-v4:

**Phase 0: Personal Catalog (before everything else — this is the seed knowledge)**

**Commit 0a: Personal catalog — CLAUDE.md audit (~150 lines)**
- `catalog/personal/README.md`
- `catalog/personal/claude-md-audit.md`

**Commit 0b: Personal catalog — settings + hooks audit (~150 lines)**
- `catalog/personal/settings-audit.md`
- `catalog/personal/hooks-audit.md`

**Commit 0c: Personal catalog — agents + skills + plugins audit (~150 lines)**
- `catalog/personal/agents-audit.md`
- `catalog/personal/skills-audit.md`
- `catalog/personal/plugins-audit.md`

**Phase 3 updated: Two skills instead of one**

**Commit 9a: Bootstrap skill (~150 lines)**
- `skills/harness-bootstrap/SKILL.md`

**Commit 9b: Update skill (~100 lines)**
- `skills/harness-update/SKILL.md`

---

## The Learning Loop

```
┌─────────────────┐
│ Joe discovers    │  "This auto-lint hook pattern
│ new pattern      │   works great for Python"
└────────┬────────┘
         ▼
┌─────────────────┐
│ Catalog it       │  Add to capability-catalog.md
│                  │  Add rule to rules-library/
│                  │  Bump catalog version
└────────┬────────┘
         ▼
┌─────────────────┐
│ /harness-update  │  Run in any existing project
│ in project X     │  "🆕 Auto-lint hook available"
│                  │  Select → apply → done
└────────┬────────┘
         ▼
┌─────────────────┐
│ /harness-update  │  Run in project Y, Z, ...
│ in project Y     │  Same new capability offered
└─────────────────┘
```

**Learn once → catalog once → propagate to many.**
