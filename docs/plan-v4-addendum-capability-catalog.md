# Plan v4 Addendum: Capability Catalog & Adaptive Discovery

> **Date**: 2026-04-06
> **Supplements**: plan-v4.md (goal, audit, repo structure remain unchanged)
> **Changes**: Replaces the fixed 8-question questionnaire with an adaptive catalog-driven discovery flow

## What's Missing from Plan v4

Plan v4 has a fixed questionnaire — 8 questions, 2 rounds, same for every project. This doesn't account for:

1. **No capability awareness** — the skill doesn't know all the harness levers available
2. **No signal-based adaptation** — detecting WebSockets should trigger different questions than detecting a CLI tool
3. **No custom generation** — if a project needs something outside the 4 examples, the skill can't help

## The Capability Catalog

A structured reference doc (`docs/capability-catalog.md`) that maps:

```
Project Signal → Relevant Capability → Question to Ask → Config to Generate
```

The skill reads this catalog at runtime and uses it to decide what questions are relevant for THIS project.

### Catalog Structure

```yaml
# Each capability entry:
capability:
  name: "Auto-format hook"
  category: hooks
  description: "Automatically format code after every edit"
  signals:                          # What triggers this capability
    - file: ".prettierrc"           # If this file exists
    - file: "pyproject.toml"        # Check for [tool.ruff] section
    - file: ".clang-format"
  question:                         # What to ask the user
    text: "Detected {formatter}. Enable auto-format on every edit?"
    options: ["Yes (recommended)", "No"]
  generates:                        # What config this produces
    target: ".claude/settings.json"
    section: "hooks.PostToolUse"
  depends_on: []                    # Other capabilities this requires
```

### Full Capability Map

#### Category: CLAUDE.md Sections

| Capability | Signals | Question | Generates |
|---|---|---|---|
| **Project identity** | Always | "Project name + description?" | CLAUDE.md header |
| **Build commands** | BUILD.bazel, package.json, pyproject.toml, pom.xml | "Detected {tool}. Correct?" | CLAUDE.md # Commands |
| **Test commands** | Test files, framework configs | "Detected {framework}. Correct?" | CLAUDE.md # Commands |
| **Lint/format commands** | .prettierrc, ruff config, .clang-format, checkstyle | "Detected {linter}/{formatter}. Correct?" | CLAUDE.md # Commands |
| **Architecture notes** | src/, lib/, pkg/ directories | "Brief architecture description?" | CLAUDE.md # Architecture |
| **Boundaries (always do)** | Build/test/format commands detected | Auto-generated from detected commands | CLAUDE.md # Boundaries |
| **Boundaries (ask first)** | Always | Pre-filled defaults, user can customize | CLAUDE.md # Boundaries |
| **Boundaries (never do)** | Always | Pre-filled defaults (push to main, skip tests, hardcode secrets) | CLAUDE.md # Boundaries |
| **Slack channel** | Global config has slack rules | "Slack channel for merge notifications?" | CLAUDE.md slack_channel |

#### Category: Settings.json

| Capability | Signals | Question | Generates |
|---|---|---|---|
| **Language permissions** | Detected language + build tool | Auto-generated (no question) | settings.json permissions.allow |
| **Credential deny list** | Always for new projects | Auto-included | settings.json permissions.deny |
| **Auto-format hook** | Formatter config detected | "Enable auto-format on edit?" | settings.json hooks.PostToolUse |
| **Auto-lint hook** | Linter config detected | "Enable auto-lint on edit?" | settings.json hooks.PostToolUse |
| **Branch protection hook** | Git repo detected | "Block edits on main branch?" | settings.json hooks.PreToolUse |
| **Stop check hook** | Always | Auto-included | settings.json hooks.Stop |

#### Category: Rules (`.claude/rules/`)

| Capability | Signals | Question | Generates |
|---|---|---|---|
| **Testing rules** | Test framework detected | "Coverage target? Test types?" | rules/testing.md |
| **Security rules** | Always | "Security level: standard/strict/minimal?" | rules/security.md |
| **Language-specific rules** | Language detected | "Include {language} idiom rules?" | rules/{lang}.md |
| **Agent team orchestration** | NOT auto-detectable | "Does this project use agent teams?" | rules/agent-teams.md |
| **Exchange conventions** | NOT auto-detectable (or: smoke_test_all.sh exists) | "Is this a trading/exchange project?" | rules/exchange.md |
| **Trading latency rules** | Exchange project = yes | Auto-included with exchange | rules/trading-latency.md |
| **API design rules** | Express/FastAPI/gRPC detected | "Include API design rules?" | rules/api-design.md |
| **Database rules** | ORM/migration files detected | "Include database migration rules?" | rules/database.md |
| **Frontend rules** | React/Vue/Svelte detected | "Include frontend component rules?" | rules/frontend.md |

#### Category: Domain-Specific (detected from project content)

| Capability | Signals | Question | Generates |
|---|---|---|---|
| **gRPC/protobuf rules** | .proto files, grpc deps | "Include proto file conventions?" | rules/grpc.md |
| **WebSocket rules** | ws/socket.io deps | "Include connection lifecycle rules?" | rules/websocket.md |
| **CI/CD awareness** | .github/workflows/, Jenkinsfile | "Include CI pipeline rules?" | CLAUDE.md boundaries |
| **Docker rules** | Dockerfile, docker-compose.yml | "Include container rules?" | rules/docker.md |
| **Terraform/IaC rules** | .tf files, CDK | "Include infrastructure rules?" | rules/iac.md |

### Catalog Extensibility

The catalog is a markdown file. To add a new capability:
1. Add an entry to the capability table
2. Optionally add an example to `examples/` or a rule to `rules-library/`
3. The skill picks it up on next run — no code changes needed

For capabilities NOT in the catalog (the "Tier 3" fallback):
- The skill says "I don't have a pre-built config for {detected pattern}. Want me to generate custom rules based on what I see in the project?"
- The LLM generates rules on the fly using its knowledge + the project context

---

## Adaptive Discovery Flow

Replaces the fixed 8-question questionnaire:

```
/harness-bootstrap
         │
         ▼
┌──────────────────────────────────┐
│ 1. Scan project                  │
│    Detect all signals:           │
│    files, deps, configs,         │
│    frameworks, patterns          │
└────────────┬──��──────────────────┘
             ▼
┌─��────────────────────────────────┐
│ 2. Read capability catalog       │  Match signals → capabilities
│    ($HARNESS_DIR/docs/           │  Build list of relevant
│     capability-catalog.md)       │  capabilities for THIS project
└────���───────┬─────────────────────┘
             ▼
��───��──────────────────────────────��
│ 3. Read global config            │  Identify inherited rules
│    (~/.claude/CLAUDE.md)         │  (skip questions already answered)
└──────���─────┬───────────��─────────┘
             ▼
┌───────────────────────────────��──┐
│ 4. Present discovery summary     │  "For your Python + FastAPI +
│    + ask adaptive questions      │   PostgreSQL project, I recommend:
│                                  │   ✅ Auto-format (ruff detected)
│    Only ask questions for        │   ✅ Testing rules (pytest detected)
│    capabilities that need        │   ❓ API design rules (FastAPI detected)
│    user input                    │   ❓ Database rules (SQLAlchemy detected)
│                                  │   ❓ Security level?
│                                  │   ❓ Coverage target?"
└──────────��─┬─────────────────────┘
             ▼
┌──────────────────���───────────────┐
│ 5. Check for unmatched signals   │  "I also detected WebSocket deps
│    (Tier 3 — custom generation)  │   but don't have pre-built rules.
��                                  │   Want me to generate custom rules?"
└────────────��─────────────────���───┘
             ▼
┌─��────────────────────────────────┐
│ 6. Read matching examples        │  Use closest example + rules-library
│    + rules-library entries       │  as reference for generation
└──────���─────┬────────────────────��┘
             ▼
┌────���─────��───────────────────────┐
│ 7. Generate .claude/ files       │  Show preview, confirm
└─────���──────┬─────────────────────┘
             ▼
┌───���──────────────────────────────┐
│ 8. Save config + update catalog  │  If custom rules generated,
│                                  │  offer to add to rules-library
└──────────────────────────────────┘
```

### Key Differences from v4's Fixed Questionnaire

| Aspect | v4 Fixed | v4 + Catalog |
|---|---|---|
| Questions | Always 8, always the same | Varies by project (3-12 depending on signals) |
| Adaptation | None — same flow for every project | Branches based on detected frameworks, deps, patterns |
| Coverage | 4 language examples only | 20+ capabilities, extensible |
| Custom needs | Not supported | Tier 3: on-the-fly generation from project context |
| Discovery UX | "Answer these questions" | "Here's what I found, here's what I recommend, confirm/adjust" |

### Example: What Discovery Looks Like

**For a Python FastAPI project with PostgreSQL:**

```
📋 Project Analysis: my-api-service
━━━━━━━━━━━━���━━━━━━━━━━━━━━━━━━━━━

Detected:
  Language:    Python 3.12
  Build:       uv
  Test:        pytest (tests/ directory)
  Formatter:   ruff (pyproject.toml)
  Linter:      ruff (pyproject.toml)
  Framework:   FastAPI (requirements)
  Database:    SQLAlchemy + Alembic (migrations/)
  API:         REST + OpenAPI (FastAPI auto-generates)

Inherited from global config:
  ✅ TDD required
  ✅ Conventional commits
  ✅ 200-line commit target
  ✅ Worktree isolation

Recommended capabilities:
  ✅ Auto-format hook (ruff detected)          — auto-included
  ✅ Testing rules (pytest detected)            — auto-included
  ✅ Security rules                             — auto-included
  ❓ API design rules (FastAPI detected)        — include?
  ❓ Database migration rules (Alembic detected) — include?
  ❓ Coverage target                            — 80% / 90% / none?
  ❓ Security level                             — standard / strict?
```

The user confirms/adjusts, and the skill generates `.claude/` accordingly.

**For a simple Python script project:**

```
📋 Project Analysis: quick-tool
━━━━━━━���━━━━━━━━━━━━━━━━━━━━━━━━━━

Detected:
  Language:    Python 3.12
  Build:       uv
  Test:        None detected
  Formatter:   None detected

Inherited from global config:
  ✅ TDD required
  ✅ Conventional commits

Recommended capabilities:
  ✅ Basic testing rules                        — auto-included
  ❓ Add ruff for formatting?                   — yes / no?
  ❓ Coverage target                            — 80% / 90% / none?

Minimal setup — 2 questions.
```

---

## Updated Repo Structure

Only change from plan-v4:

```diff
 harness/
 ├─��� docs/
+│   ├── capability-catalog.md        # NEW: signal→capability mapping
 │   ├── harness-anatomy.md
 │   ├── hooks-cookbook.md
 │   ├── rules-cookbook.md
 │   ├── compatibility-guide.md
 │   └── global-audit.md
 ├── examples/
 │   └── (unchanged)
 ├── rules-library/
 │   ├── agent-teams.md
 │   ├── exchange.md
 │   ├── trading-latency.md
+│   ├── api-design.md                # NEW: for FastAPI/Express/gRPC projects
+│   ├── database.md                  # NEW: for ORM/migration projects
+│   └── frontend.md                  # NEW: for React/Vue/Svelte projects
 ├── configs/
 └── skill/
     └── SKILL.md
```

### Updated Implementation Plan

Add these commits to plan-v4:

**Insert after Commit 4 (Rules cookbook):**

**Commit 4.5: Capability catalog (~150 lines)**
- `docs/capability-catalog.md` — the full signal→capability→question→config mapping

**Insert after Commit 5 (Rules library):**

**Commit 5.5: Extended rules library (~120 lines)**
- `rules-library/api-design.md`
- `rules-library/database.md`
- `rules-library/frontend.md`

**Commit 9 (Bootstrap skill) updated:**
- SKILL.md now reads `capability-catalog.md` and adapts questions based on detected signals
- Includes Tier 3 fallback for unmatched signals

---

## What This Gives Us

The capability catalog is the **knowledge backbone** of the harness project. It:

1. **Documents all available configurations** — anyone can browse it to understand what harness options exist
2. **Drives the adaptive questionnaire** — the skill reads it to decide what to ask
3. **Is extensible** — add a new capability by adding a row to the catalog + optionally a rule to rules-library
4. **Supports Tier 3 custom generation** — for signals not in the catalog, the skill knows to fall back to on-the-fly generation
5. **Works like superpowers:brainstorming** — discovery-first, recommendations-second, generation-third
