# Harness Knowledge Base — Plan v5 (Consolidated)

> **Status**: Final draft
> **Date**: 2026-04-06
> **Consolidates**: plan-v4.md + capability-catalog addendum + personal-catalog addendum
> **Plan evolution**: v1 (over-engineered) → v2 (narrowed) → v3 (project-only) → v4 (context segregation) → v5 (consolidated + implementation plan)

---

## The Problem

Joe's global `~/.claude/CLAUDE.md` is 224 lines. Every project loads all 224 lines — exchange smoke tests pollute Python CLIs, 60 lines of agent team orchestration load in single-agent sessions, trading-specific rules appear in study projects.

**Goal**: Trim global to ~90-100 lines of truly universal rules. Move project-specific rules to project-level `.claude/` configs. Build tooling to do this efficiently at scale across 17+ projects.

```
BEFORE: ~/.claude/CLAUDE.md (224 lines → every project)
AFTER:  ~/.claude/CLAUDE.md (~90 lines → universal only)
        + each project's .claude/ (project-specific only)
```

---

## What This Repo Is

A **knowledge base, capability catalog, and tooling** for Claude Code harness configuration:

1. **Personal catalog** (`catalog/personal/`) — Joe's existing config documented with reasoning
2. **Capability catalog** (`docs/capability-catalog.md`) — all available harness capabilities mapped to project signals
3. **Rules library** (`rules-library/`) — reusable rule files extracted from global config
4. **Examples** (`examples/`) — complete reference configs the LLM reads when generating
5. **Cookbooks** (`docs/`) — practical recipes for hooks, rules, compatibility
6. **Two skills**:
   - `/harness-bootstrap` — initial project setup (scan, discover, generate)
   - `/harness-update` — incremental updates (diff catalog vs installed, apply new capabilities)

### Relationship to Existing Tools

```
Blank/existing repo          Existing codebase           Ongoing development
needing .claude/ config      needing optimization        
         │                          │                          │
         ▼                          ▼                          ▼
/harness-bootstrap          /claude-automation-        superpowers skills
/harness-update              recommender               (brainstorming, plans,
                             (read-only advisory)       TDD, etc.)
```

---

## Global CLAUDE.md Audit

### KEEP GLOBAL (~90 lines)

| Section | Lines | Why |
|---|---|---|
| Build System table | 3-17 | Language→tool mapping — universal |
| Project Planning | 18-24 | Researcher review — all non-trivial projects |
| Testing | 25-39 | TDD, coverage, failure injection — universal philosophy |
| Documentation | 41-55 | README/docs sync — universal |
| Scope | 56-57 | Universal |
| Security | 59-68 | OWASP, crypto, secrets — universal |
| What the Agent Gets Wrong | 70-76 | Universal guardrails |
| Commit Rules + format | 98-121 | Universal commit discipline |
| Model Selection | 123-141 | Universal preference (trim "human review mandatory" to project-level) |
| Error Recovery | 142-148 | Universal |
| Context Management | 149-153 | Universal |
| Session Memory | 218-224 | Universal |

### MOVE TO PROJECT LEVEL (~95 lines)

| Section | Lines | Destination |
|---|---|---|
| Agent Team Orchestration + Swim Lanes | 155-215 (60 lines) | `rules-library/agent-teams.md` → project `.claude/rules/` |
| Exchange Smoke Test Gate | 94-96 | `rules-library/exchange.md` → exchange projects only |
| Slack Notifications | 78-81 | Per-project CLAUDE.md (`slack_channel`) |
| Branching Strategy details | 83-92 | Core "never commit to main" stays global; worktree details → project |

**Net**: 224 → ~90-100 lines (55% reduction)

---

## Repo Structure

```
harness/
├── README.md
├── catalog/
│   └── personal/                       # Seed knowledge — Joe's config with reasoning
│       ├── README.md                   # Config philosophy
│       ├── claude-md-audit.md          # Each CLAUDE.md section: what/why/when
│       ├── settings-audit.md           # Each settings.json entry: what/why/when
│       ├── agents-audit.md             # Each agent: purpose, model, when to use
│       ├── hooks-audit.md              # Each hook: what it enforces, why
│       ├── skills-audit.md             # Each skill: purpose, invocation
│       └── plugins-audit.md            # Each plugin: what it provides, why enabled
├── docs/
│   ├── capability-catalog.md           # Signal → capability → question → config mapping
│   ├── harness-anatomy.md              # Scopes, precedence, @imports, file roles
│   ├── hooks-cookbook.md                # Copy-paste hook recipes
│   ├── rules-cookbook.md                # Testing, security, language rule patterns
│   ├── compatibility-guide.md          # Global vs project interactions
│   └── global-audit.md                 # What stays global vs moves to project
├── examples/                           # Complete reference configs for LLM generation
│   ├── python-cli/                     # Python + uv + pytest + ruff
│   │   ├── claude-md.example.md
│   │   ├── settings.example.json
│   │   └── rules/
│   ├── typescript-web/                 # TypeScript + npm + vitest + prettier
│   │   ├── claude-md.example.md
│   │   ├── settings.example.json
│   │   └── rules/
│   ├── cpp-bazel/                      # C++ + Bazel + gtest + clang-format
│   │   ├── claude-md.example.md
│   │   ├── settings.example.json
│   │   └── rules/
│   └── java-bazel/                     # Java + Bazel + JUnit
│       ├── claude-md.example.md
│       ├── settings.example.json
│       └── rules/
├── rules-library/                      # Reusable rules (copied into projects)
│   ├── agent-teams.md                  # Orchestration (extracted from global)
│   ├── exchange.md                     # Smoke test + exchange conventions
│   ├── trading-latency.md              # Latency budgets, hot path rules
│   ├── api-design.md                   # FastAPI/Express/gRPC conventions
│   ├── database.md                     # ORM/migration rules
│   └── frontend.md                     # React/Vue/Svelte rules
├── configs/                            # Saved bootstrap answers (JSON)
│   └── .gitkeep
└── skills/
    ├── harness-bootstrap/
    │   └── SKILL.md                    # Initial project setup
    └── harness-update/
        └── SKILL.md                    # Incremental capability updates
```

---

## Capability Catalog (`docs/capability-catalog.md`)

Maps: `Project Signal → Capability → Question → Config Generated`

The skill reads this at runtime to adapt its questions per project.

### CLAUDE.md Capabilities

| Capability | Signal | Question | Generates |
|---|---|---|---|
| Project identity | Always | "Name + description?" | CLAUDE.md header |
| Build commands | BUILD.bazel, package.json, pyproject.toml | "Detected {tool}. Correct?" | # Commands |
| Test commands | Test files, framework configs | "Detected {framework}. Correct?" | # Commands |
| Lint/format commands | .prettierrc, ruff, .clang-format | "Detected {tool}. Correct?" | # Commands |
| Architecture notes | src/, lib/, pkg/ dirs | "Brief architecture?" | # Architecture |
| Boundaries | Always | Pre-filled from detected commands | # Boundaries |
| Slack channel | Global has slack rules | "Channel?" | slack_channel |

### Settings.json Capabilities

| Capability | Signal | Question | Generates |
|---|---|---|---|
| Language permissions | Language + build tool | Auto-generated | permissions.allow |
| Credential deny list | Always | Auto-included | permissions.deny |
| Auto-format hook | Formatter config | "Enable auto-format?" | hooks.PostToolUse |
| Auto-lint hook | Linter config | "Enable auto-lint?" | hooks.PostToolUse |
| Branch protection | Git repo | "Block edits on main?" | hooks.PreToolUse |
| Stop check | Always | Auto-included | hooks.Stop |

### Rules Capabilities

| Capability | Signal | Question | Generates |
|---|---|---|---|
| Testing rules | Test framework | "Coverage? Test types?" | rules/testing.md |
| Security rules | Always | "Standard/strict/minimal?" | rules/security.md |
| Language rules | Language detected | "Include {lang} rules?" | rules/{lang}.md |
| Agent teams | NOT auto-detectable | "Uses agent teams?" | rules/agent-teams.md |
| Exchange | smoke_test_all.sh or manual | "Trading/exchange project?" | rules/exchange.md |
| Trading latency | Exchange = yes | Auto-included | rules/trading-latency.md |
| API design | FastAPI/Express/gRPC | "Include API rules?" | rules/api-design.md |
| Database | ORM/migrations detected | "Include DB rules?" | rules/database.md |
| Frontend | React/Vue/Svelte | "Include frontend rules?" | rules/frontend.md |
| gRPC/protobuf | .proto files | "Include proto rules?" | rules/grpc.md |
| CI/CD | .github/workflows/ | "Include CI rules?" | CLAUDE.md boundaries |
| Docker | Dockerfile | "Include container rules?" | rules/docker.md |

### Three Tiers

1. **Auto-included** — signal detected, no question needed (permissions, deny list, stop check)
2. **Confirm/adjust** — signal detected, user confirms (build/test/format commands, coverage target)
3. **Custom on-the-fly** — no catalog match, LLM generates from project context

---

## `/harness-bootstrap` Skill

### Adaptive Discovery Flow

```
/harness-bootstrap in project directory
         │
    ┌────▼─────────────────┐
    │ 1. Scan project       │  Detect language, build, test, deps,
    │                       │  frameworks, config files
    └────┬─────────────────-┘
    ┌────▼─────────────────┐
    │ 2. Read catalog       │  Match signals → relevant capabilities
    │    + global config    │  Identify inherited rules (skip asking)
    └────┬─────────────────-┘
    ┌────▼─────────────────┐
    │ 3. Check saved configs│  Offer reuse if matching config exists
    └────┬──────────────────┘
    ┌────▼─────────────────┐
    │ 4. Present discovery  │  "Detected: Python + FastAPI + Alembic
    │    + adaptive Qs      │   ✅ auto-format (ruff) — auto-included
    │                       │   ❓ API rules? DB rules? Coverage?"
    └────┬─────────────────-┘
    ┌────▼─────────────────┐
    │ 5. Tier 3 fallback    │  "Detected WebSocket deps — no pre-built
    │                       │   rules. Generate custom?"
    └────┬─────────────────-┘
    ┌────▼─────────────────┐
    │ 6. Read example +     │  Closest example as LLM reference
    │    rules-library      │
    └────┬─────────────────-┘
    ┌────▼─────────────────┐
    │ 7. Generate .claude/  │  Preview → confirm → write
    └────┬─────────────────-┘
    ┌────▼─────────────────┐
    │ 8. Save config        │  To harness/configs/{name}.json
    └───────────────────────┘
```

---

## `/harness-update` Skill

### Incremental Update Flow

```
/harness-update in existing project
         │
    ┌────▼─────────────────┐
    │ 1. Read current       │  What's in .claude/ now?
    │    project config     │  What capabilities installed?
    └────┬─────────────────-┘
    ┌────▼─────────────────┐
    │ 2. Read catalog       │  What capabilities exist?
    │    + scan for signals  │  What's new since last sync?
    └────┬─────────────────-┘
    ┌────▼─────────────────┐
    │ 3. Diff               │  Catalog vs installed
    └────┬─────────────────-┘
    ┌────▼─────────────────┐
    │ 4. Present update     │  🆕 New: gRPC rules, auto-lint v2
    │    menu               │  📦 Updated: testing v2→v3
    │                       │  ✅ Current: exchange, agent-teams
    └────┬─────────────────-┘
    ┌────▼─────────────────┐
    │ 5. Apply selected     │  Show diff → confirm → write
    └────┬─────────────────-┘
    ┌────▼─────────────────┐
    │ 6. Update version     │  Record catalog version in
    │    marker             │  .claude/settings.local.json
    └───────────────────────┘
```

### Version Tracking

`.claude/settings.local.json` (gitignored):
```json
{
  "harness": {
    "catalog_version": "5",
    "last_synced": "2026-04-06",
    "installed_capabilities": ["testing-rules:v3", "security-rules:v2", "exchange:v1"]
  }
}
```

### The Learning Loop

```
Discover new pattern → Catalog it → /harness-update in any project → Applied
```

**Learn once → catalog once → propagate to many.**

---

## Implementation Plan

### Phase 0: Personal Catalog (seed knowledge)

All work in a feature branch via worktree.

**Commit 1: Catalog scaffold + CLAUDE.md audit (~150 lines)**
- `catalog/personal/README.md` (~20 lines)
- `catalog/personal/claude-md-audit.md` (~130 lines)
  - Every section of `~/.claude/CLAUDE.md` documented with what/why/when/candidate-for

**Commit 2: Settings + hooks audit (~150 lines)**
- `catalog/personal/settings-audit.md` (~100 lines)
  - Permissions, env vars, MCP servers, feature flags
- `catalog/personal/hooks-audit.md` (~50 lines)
  - Notification, PermissionRequest, Stop hooks

**Commit 3: Agents + skills + plugins audit (~150 lines)**
- `catalog/personal/agents-audit.md` (~80 lines)
  - 8 agents: purpose, model, when to use, when NOT
- `catalog/personal/skills-audit.md` (~30 lines)
  - commit, pr skills
- `catalog/personal/plugins-audit.md` (~40 lines)
  - superpowers, code-review, frontend-design, slack, financial, etc.

### Phase 1: Foundation Docs

**Commit 4: README + global audit (~120 lines)**
- `README.md` (~60 lines) — what this repo is, when to use vs alternatives
- `docs/global-audit.md` (~60 lines) — the keep/move/borderline tables

**Commit 5: Harness anatomy + compatibility guide (~180 lines)**
- `docs/harness-anatomy.md` (~100 lines) — scopes, precedence, loading, @imports
- `docs/compatibility-guide.md` (~80 lines) — global vs project, merging, conflicts

**Commit 6: Hooks cookbook (~120 lines)**
- `docs/hooks-cookbook.md` — auto-format, branch protection, file protection, lint, stop check, notifications, post-compaction

**Commit 7: Rules cookbook (~120 lines)**
- `docs/rules-cookbook.md` — testing, security, language packs, boundaries, path-scoped frontmatter

### Phase 2: Capability Catalog + Rules Library

**Commit 8: Capability catalog (~150 lines)**
- `docs/capability-catalog.md` — the full signal→capability���question→config map

**Commit 9: Rules library — core (~100 lines)**
- `rules-library/agent-teams.md` (~60 lines, extracted from global)
- `rules-library/exchange.md` (~20 lines)
- `rules-library/trading-latency.md` (~20 lines)

**Commit 10: Rules library — extended (~120 lines)**
- `rules-library/api-design.md` (~40 lines)
- `rules-library/database.md` (~40 lines)
- `rules-library/frontend.md` (~40 lines)

### Phase 3: Examples

**Commit 11: Python CLI example (~90 lines)**
- `examples/python-cli/claude-md.example.md`
- `examples/python-cli/settings.example.json`
- `examples/python-cli/rules/testing.example.md`
- `examples/python-cli/rules/security.example.md`

**Commit 12: C++ Bazel example (~100 lines)**
- `examples/cpp-bazel/claude-md.example.md`
- `examples/cpp-bazel/settings.example.json`
- `examples/cpp-bazel/rules/` (testing, security, cpp, exchange, agent-teams)

**Commit 13: TypeScript + Java examples (~180 lines)**
- `examples/typescript-web/` (~90 lines)
- `examples/java-bazel/` (~90 lines)

### Phase 4: Skills

**Commit 14: Bootstrap skill (~150 lines)**
- `skills/harness-bootstrap/SKILL.md`
- `configs/.gitkeep`

**Commit 15: Update skill (~100 lines)**
- `skills/harness-update/SKILL.md`

**Commit 16: Install + portability**
- Symlink: `skills/harness-bootstrap/` → `~/.claude/skills/harness-bootstrap/`
- Symlink: `skills/harness-update/` → `~/.claude/skills/harness-update/`
- Add `HARNESS_DIR` env var to `~/.claude/settings.json`

### Phase 5: Validation

**Commit 17: Validate on existing project**
- Run `/harness-bootstrap` on `network-toolkit` (simple Python project)
- Verify: correct .claude/ generated, no exchange/agent-teams rules
- Run `/harness-bootstrap` on `exchange-connectivity` (trading project)
- Verify: exchange rules + agent-teams included
- Run `/harness-update` after adding a new capability to catalog
- Verify: new capability offered and applied

### Phase 6: Global Trim (separate PR, after all validation)

- Run `/harness-bootstrap` on remaining projects
- Trim `~/.claude/CLAUDE.md` from 224 → ~90-100 lines
- Verify all projects work with lean global + project configs

---

## Verification Checklist

1. **Personal catalog complete**: Every setting documented with reasoning
2. **Capability catalog**: 20+ capabilities mapped with signals
3. **Auto-detection**: Skill correctly identifies language, build, test, formatter from project files
4. **Adaptive questions**: Different projects get different questions based on signals
5. **Tier 3 fallback**: Unmatched signals offer custom on-the-fly generation
6. **Trading project**: Gets exchange + agent-teams + latency rules
7. **Non-trading project**: Does NOT get exchange/agent-teams/latency rules
8. **Context segregation**: After global trim, irrelevant rules no longer load
9. **Update flow**: New catalog capability appears in `/harness-update` menu
10. **Version tracking**: `.claude/settings.local.json` records installed capabilities
11. **Config reuse**: Saved configs work for similar new projects
12. **Superpowers integration**: Generated config works with superpowers:writing-plans

---

## Summary

| Deliverable | Purpose | Phase |
|---|---|---|
| Personal catalog | Seed knowledge with reasoning | 0 |
| Reference docs + cookbooks | Practical harness knowledge | 1 |
| Capability catalog | Signal→config mapping, drives adaptive skill | 2 |
| Rules library | Reusable rules extracted from global | 2 |
| Examples | LLM reference for generation quality | 3 |
| `/harness-bootstrap` | Initial project setup | 4 |
| `/harness-update` | Incremental capability updates | 4 |
| Global CLAUDE.md trim | The actual goal — context segregation | 6 |

**17 commits across 6 phases. All commits under 200 lines.**

**The learning loop**: Discover → catalog → propagate. The harness repo grows smarter over time, and every project benefits.
