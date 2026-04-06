# Harness Bootstrap — Plan v4 (Context Segregation)

> **Status**: Draft
> **Date**: 2026-04-06
> **Supersedes**: plan-v3.md
> **Key reframe**: The problem isn't bootstrapping blank repos — it's trimming a bloated global CLAUDE.md and distributing rules to the right scope level.

## The Problem

Joe's global `~/.claude/CLAUDE.md` is 224 lines. Every project — Python CLI, React web app, C++ trading system, learning notes — loads all 224 lines. This causes:

1. **Context pollution**: Exchange smoke test rules appear when editing a knowledge graph. Agent team orchestration (60 lines) loads in single-agent sessions. Trading-specific merge strategies appear in Python study projects.
2. **Wasted context window**: Every line competes for attention. Anthropic recommends <200 lines. HumanLayer recommends <60. At 224 lines, irrelevant rules dilute important ones.
3. **No project identity**: Projects rely on the global config instead of having their own. 14 of 17 projects have zero Claude configuration.

## The Goal

**Before:**
```
~/.claude/CLAUDE.md (224 lines — everything)
    ↓ loaded into every session
    ├── python-cli project (gets exchange smoke test rules)
    ├── knowledge-graph project (gets swim lane tracking)
    ├── market-data-feedhandler (gets npm rules)
    └── learning-plan (gets agent orchestration)
```

**After:**
```
~/.claude/CLAUDE.md (~80-100 lines — truly universal only)
    ↓ loaded into every session (lean, relevant)

Each project gets its own .claude/:
    ├── exchange-connectivity/.claude/CLAUDE.md (exchange-specific rules)
    ├── knowledge-graph/.claude/CLAUDE.md (React/TS-specific rules)
    ├── market-data-feedhandler/.claude/CLAUDE.md (C++/Bazel-specific rules)
    └── learning-plan/.claude/CLAUDE.md (minimal — study project)
```

## Global CLAUDE.md Audit

Current 224 lines categorized by where they belong:

### KEEP GLOBAL (~90 lines) — Truly universal rules

| Section | Lines | Why global |
|---|---|---|
| Build System table | 3-17 | Language→tool mapping applies everywhere |
| Testing (TDD, coverage, failure injection) | 25-39 | Universal testing philosophy |
| Documentation (README, docs/) | 41-55 | Universal doc standards |
| Scope | 56-57 | Universal |
| Security | 59-68 | Universal security rules |
| What the Agent Gets Wrong | 70-76 | Universal guardrails |
| Commit Rules + format | 98-121 | Universal commit discipline |
| Error Recovery | 142-148 | Universal |
| Context Management | 149-153 | Universal |
| Session Memory | 218-224 | Universal |

### MOVE TO PROJECT LEVEL (~95 lines) — Project/workflow-specific

| Section | Lines | Where it should go |
|---|---|---|
| Agent Team Orchestration | 155-215 (60 lines!) | Only projects that use agent teams. Could be a `.claude/rules/agent-teams.md` included per-project. |
| Swim Lane Tracking | 199-215 | Part of agent teams — same destination |
| Exchange Smoke Test Gate | 94-96 | Only `exchange-connectivity/` and `agency-trading/` |
| Slack Notifications | 78-81 | Per-project (channel varies). Project CLAUDE.md specifies `slack_channel`. |
| Branching Strategy (worktree details) | 83-92 | Core rule (never commit to main) stays global. Worktree details move to projects that use them. |
| Model Selection table | 123-141 | Could stay global (universal preference) or move to project level. Borderline. |

### BORDERLINE (~35 lines) — Could go either way

| Section | Lines | Decision |
|---|---|---|
| Project Planning (researcher review) | 18-24 | Keep global — applies to all non-trivial projects |
| Model Selection | 123-141 | Keep global — universal preference, but trim the "human review mandatory" part to project-level for production projects only |

### Net result
- Global: **~90-100 lines** (down from 224) — 55% reduction
- Freed: **~95 lines** of project/workflow-specific rules moved to where they belong

---

## What the Harness Bootstrap Does

Two modes:

### Mode 1: Migrate Existing Project
For each of Joe's 17 existing projects:
1. Scan project directory — detect language, build system, test framework, existing config
2. Read global CLAUDE.md — identify which global rules are relevant to THIS project
3. Generate `.claude/CLAUDE.md` with project-specific commands, boundaries, and any rules migrated from global
4. Generate `.claude/settings.json` with language-appropriate permissions and hooks
5. Generate `.claude/rules/` for testing, security, and optionally language-specific rules
6. If the project uses agent teams: include agent orchestration rules
7. If the project is an exchange implementation: include smoke test gate

### Mode 2: New Project Setup
Same questionnaire but informed by the now-lean global config. Fewer questions needed because the global config covers the universal rules.

---

## The Bootstrap Skill (`/harness-bootstrap`)

### Flow

```
User runs /harness-bootstrap in a project directory
         │
         ▼
┌──────────────────────────────────┐
│ 1. Scan project                  │ Detect: language, build files,
│    (Glob, Grep, Read)            │ test framework, package manager,
│                                  │ existing .claude/ config
└────────────┬─────────────────────┘
             ▼
┌──────────────────────────────────┐
│ 2. Read global config            │ Identify inherited universal rules.
│    ~/.claude/CLAUDE.md           │ Identify rules that should migrate
│                                  │ to this project specifically.
└────────────┬─────────────────────┘
             ▼
┌──────────────────────────────────┐
│ 3. Check for saved configs       │ "Found saved config for python-cli.
│    in $HARNESS_DIR/configs/      │  Use as starting point?"
└────────────┬─────────────────────┘
             ▼
┌──────────────────────────────────┐
│ 4. Present findings + ask        │ "Detected: Python, uv, pytest.
│    remaining questions           │  Is this a trading project? (y/n)
│    (1-2 rounds)                  │  Security level? Coverage target?"
└────────────┬─────────────────────┘
             ▼
┌──────────────────────────────────┐
│ 5. Read matching example         │ Use closest example as reference
│    from $HARNESS_DIR/examples/   │ for generation quality
└────────────┬─────────────────────┘
             ▼
┌──────────────────────────────────┐
│ 6. Generate .claude/ files       │ Adapted from example + scan +
│    (show preview, confirm)       │ answers. Prompt before overwrite.
└────────────┬─────────────────────┘
             ▼
┌──────────────────────────────────┐
│ 7. Save config for reuse         │
└──────────────────────────────────┘
```

### Key difference from v3: Auto-detection replaces most questions

The skill scans the project and auto-detects:
- Language (from file extensions, build files)
- Build tool (from BUILD.bazel, package.json, pyproject.toml, pom.xml)
- Test framework (from test file patterns, config files)
- Formatter/linter (from .prettierrc, pyproject.toml [ruff], .clang-format)
- Package manager (from lock files)

This means the questionnaire shrinks to only things that CAN'T be auto-detected:

**Round 1 — Confirm detections + project identity:**

| # | Question | Type | Options |
|---|---|---|---|
| 1 | "Detected Python + uv + pytest. Correct?" | Single | Yes / Correct with changes |
| 2 | Project name + one-line description | Free text | (Other) |
| 3 | Is this a trading/exchange project? | Single | Yes (include smoke test + latency rules) / No |
| 4 | Uses agent teams? | Single | Yes (include orchestration rules) / No |

**Round 2 — Preferences (only non-inherited, non-detectable):**

| # | Question | Type | Options |
|---|---|---|---|
| 5 | Coverage target | Single | 80%, 90%, No target |
| 6 | Security level | Single | Standard (OWASP), Strict, Minimal |
| 7 | Include language-specific rules? | Single | Yes, No |
| 8 | Slack notification channel | Single | #general, Custom, None |

**Total: ~8 questions in 2 rounds.** Most answers are confirmations of auto-detected values.

### What Gets Generated

**For a Python CLI project (non-trading, no agent teams):**

```
project-root/
├── .claude/
│   ├── CLAUDE.md              # ~40 lines: commands, boundaries, testing
│   ├── settings.json          # Permissions + auto-format hook
│   └── rules/
│       ├── testing.md         # Testing rules (from global, adapted)
│       ├── security.md        # Security checklist
│       └── python.md          # Python-specific (if opted in)
└── .gitignore                 # Updated with CLAUDE.local.md etc.
```

**For an exchange-connectivity project (trading, agent teams):**

```
project-root/
├── .claude/
│   ├── CLAUDE.md              # ~60 lines: commands, boundaries, exchange-specific
│   ├── settings.json          # Permissions + hooks
│   └── rules/
│       ├── testing.md         # Testing + latency benchmarks
│       ├── security.md        # Strict security
│       ├── cpp.md             # C++ idioms
│       ├── agent-teams.md     # Orchestration rules (migrated from global)
│       └── exchange.md        # Smoke test gate, exchange conventions
└── .gitignore
```

**The agent-teams.md rule** (migrated from global, ~60 lines) only loads when working in projects that opt in. No more polluting every session.

**The exchange.md rule** (smoke test gate + exchange conventions) only loads in exchange projects. No more showing up in Python study projects.

---

## Repo Structure

```
harness/
├── README.md                           # What, why, when to use vs alternatives
├── docs/
│   ├── plan-v1.md ... plan-v4.md       # Plan evolution (archived)
│   ├── harness-anatomy.md              # Scopes, precedence, @imports
│   ├── hooks-cookbook.md                # Copy-paste hook recipes
│   ├── rules-cookbook.md                # Testing, security, language rules
│   ├── compatibility-guide.md          # Global vs project interactions
│   └── global-audit.md                 # Analysis of what belongs at each scope
├── examples/                           # Reference configs the LLM reads when generating
│   ├── python-cli/                     # Python + uv + pytest
│   │   ├── claude-md.example.md
│   │   ├── settings.example.json
│   │   └── rules/
│   ├── typescript-web/                 # TypeScript + npm + vitest
│   │   ├── claude-md.example.md
│   │   ├── settings.example.json
│   │   └── rules/
│   ├── cpp-bazel/                      # C++ + Bazel + gtest
│   │   ├── claude-md.example.md
│   │   ├── settings.example.json
│   │   └── rules/
│   └── java-bazel/                     # Java + Bazel + JUnit
│       ├── claude-md.example.md
│       ├── settings.example.json
│       └── rules/
├── rules-library/                      # Reusable rule files (copy into projects)
│   ├── agent-teams.md                  # Agent orchestration (60 lines from global)
│   ├── exchange.md                     # Exchange smoke test + conventions
│   └── trading-latency.md              # Latency budgets, hot path rules
├── configs/                            # Saved bootstrap answers
│   └── .gitkeep
└── skill/
    └── SKILL.md                        # /harness-bootstrap
```

### New: `rules-library/`
Reusable rule files extracted from the global CLAUDE.md. These are the rules that move from global to project-level. The bootstrap skill copies relevant ones into `.claude/rules/` during generation.

---

## Implementation Plan

### Phase 1: Foundation (commits 1-4)

**Commit 1: Repo scaffold + README + global audit (~120 lines)**
- `README.md` — what this repo is, the context segregation problem, when to use
- `docs/global-audit.md` — the audit table from this plan showing what stays/moves

**Commit 2: Reference docs (~180 lines)**
- `docs/harness-anatomy.md` (~100 lines)
- `docs/compatibility-guide.md` (~80 lines)

**Commit 3: Hooks cookbook (~120 lines)**
- `docs/hooks-cookbook.md`

**Commit 4: Rules cookbook (~120 lines)**
- `docs/rules-cookbook.md`

### Phase 2: Examples + Rules Library (commits 5-8)

**Commit 5: Rules library (~100 lines)**
- `rules-library/agent-teams.md` (extracted from global lines 155-215)
- `rules-library/exchange.md` (extracted from global lines 94-96 + expanded)
- `rules-library/trading-latency.md` (new, for trading projects)

**Commit 6: Example — Python CLI (~90 lines)**
- `examples/python-cli/` (claude-md, settings, rules/)

**Commit 7: Example — C++ Bazel (~100 lines)**
- `examples/cpp-bazel/` (claude-md, settings, rules/ including exchange + agent-teams)

**Commit 8: Examples — TypeScript + Java (~180 lines)**
- `examples/typescript-web/` (~90 lines)
- `examples/java-bazel/` (~90 lines)

### Phase 3: Skill + Migration (commits 9-11)

**Commit 9: Bootstrap skill (~150 lines)**
- `skill/SKILL.md`
- `configs/.gitkeep`

**Commit 10: Install + env var**
- Symlink: `ln -s $PWD/skill ~/.claude/skills/harness-bootstrap`
- Add `HARNESS_DIR` to `~/.claude/settings.json` env

**Commit 11: Validation**
- Run `/harness-bootstrap` on one existing project (e.g., `network-toolkit`)
- Verify generated `.claude/` is correct
- Verify the project works with the new project-level config
- Test that irrelevant global rules (exchange, agent teams) do NOT appear

### Phase 4: Global CLAUDE.md Trim (separate PR, after validation)

**This happens AFTER the bootstrap is validated**, not during:
1. Run `/harness-bootstrap` on key projects to give them project-level configs
2. Trim `~/.claude/CLAUDE.md` — remove sections that are now in project configs
3. Verify all projects still work correctly with the leaner global + their project configs

---

## Verification Plan

1. **Auto-detection works**: Skill correctly identifies language, build tool, test framework from project files
2. **Questions are minimal**: Only 2 rounds (~8 questions), most are confirmations
3. **Example reading**: Skill reads matching example and generates adapted output
4. **Project generation**: `.claude/CLAUDE.md`, `settings.json`, `rules/` created correctly
5. **Trading project**: Exchange project gets `rules/exchange.md` + `rules/agent-teams.md`
6. **Non-trading project**: Python CLI does NOT get exchange or agent-teams rules
7. **Context segregation**: After trimming global, non-trading projects no longer see exchange rules
8. **Config roundtrip**: Saved config can be reused on similar project
9. **Compatibility**: Generated project config doesn't contradict trimmed global config
10. **Superpowers**: Generated project works with superpowers:writing-plans flow

---

## Future Enhancements

- Monorepo subdirectory config generation
- Update/upgrade mode (re-run, apply diff)
- Multi-language project support
- Go and Rust examples
- Integration with claude-automation-recommender (run after bootstrap to refine)
