# Project Harness Bootstrap Skills — Plan v1

> **Status**: Draft — pending review
> **Date**: 2026-04-05
> **Author**: Joe + Claude

## Context

Joe has a sophisticated global Claude Code configuration (`~/.claude/CLAUDE.md`, agents, skills, hooks, settings) optimized for trading systems and multi-agent workflows. He wants a **reusable set of skills** that can bootstrap any new blank repo with the right harness — CLAUDE.md, hooks, skills, settings, project structure — tailored to the specific project's needs.

The goal: run `/harness-bootstrap` on a new repo, answer questions interactively, and get a fully configured harness that enforces: **requirement → research → plan → task decomposition → implementation → testing → validation → repeat**.

## Design Decisions (User-Confirmed)

| Decision | Choice |
|---|---|
| UX style | Interactive first run, saves config to `~/.claude/harness-configs/` for reuse |
| Skill scope | **Global core** (`~/.claude/skills/`) + project overrides via CLAUDE.md |
| Cycle autonomy | Pause only for plan approval; research/decompose/implement/validate are autonomous |
| Rules | Agnostic core + optional language-specific packs |
| Agents | Use global agents; generate project-specific only if user identifies a gap |
| Config storage | `~/.claude/harness-configs/{name}.yaml` (global library) |
| Conflict handling | Flag each conflict, user decides per-conflict |
| Validation | "Hello World" CLI command test run |

---

## Architecture Overview

```
                     ┌─────────────────────────────────┐
                     │   /harness-bootstrap (skill)     │
                     │   Interactive questionnaire       │
                     │   + Compatibility checker         │
                     │   + Config save/load              │
                     └──────────┬──────────────────┬────┘
                                │ generates         │ saves
                    ┌───────────▼────────┐   ┌─────▼──────────────┐
                    │ Project .claude/   │   │ ~/.claude/          │
                    │  CLAUDE.md         │   │  harness-configs/   │
                    │  settings.json     │   │   python-cli.yaml   │
                    │  rules/            │   │   cpp-trading.yaml  │
                    │   testing.md       │   │                     │
                    │   security.md      │   └─────────────────────┘
                    │   {lang}.md (opt)  │
                    │  skills/ (overrides│
                    │   only if needed)  │
                    └────────────────────┘

    Global workflow skills (always available, not generated per-project):
    ┌──────────┐  ┌──────┐  ┌───────────┐  ┌───────────┐  ┌──────────┐
    │/research │→ │/plan │→ │/decompose │→ │/implement │→ │/validate │
    └──────────┘  └──┬───┘  └───────────┘  └───────────┘  └──────────┘
                     │ ⏸ human approval
    ┌────────────────┴───────────────────────────────────────────────┐
    │  /cycle  — orchestrates all 5 skills end-to-end               │
    │  Pauses only for plan approval. Loops on validation failure.  │
    └───────────────────────────────────────────────────────────────┘
```

---

## What Gets Created

### A. Global Skills (`~/.claude/skills/`) — 7 skills

These are **universal workflow skills** that work with any project. They read project-specific config from the project's CLAUDE.md at runtime.

#### 1. `harness-bootstrap/SKILL.md` (~120 lines)
The main entry point. User-invocable as `/harness-bootstrap`.

**Flow:**
1. Check if `~/.claude/harness-configs/` has saved configs → offer to reuse
2. If new: ask Category 1-7 questions via `AskUserQuestion` (4 questions at a time)
3. Run compatibility checker against `~/.claude/CLAUDE.md`
4. Flag conflicts → user resolves
5. Generate project files (CLAUDE.md, settings.json, rules/)
6. Save answers to `~/.claude/harness-configs/{project-name}.yaml`
7. Run validation: create a "Hello World" feature using `/cycle`

**Questionnaire categories** (asked in batches of 4 via AskUserQuestion):

**Batch 1 — Identity & Build:**
- Project name + description
- Primary language(s)
- Build tool
- Monorepo or standalone?

**Batch 2 — Testing:**
- TDD or test-after?
- Test framework (auto-detect or specify)
- Coverage target
- Test types required (multi-select: unit, integration, e2e, perf, failure injection, property-based, smoke)

**Batch 3 — Planning & Quality:**
- Plan detail level (light/medium/detailed)
- Task granularity (per-commit / per-feature)
- Commit line limit
- Code review gate (human / agent / hybrid)

**Batch 4 — Workflow & Docs:**
- Git worktrees? (yes/no)
- Merge strategy
- Slack channel for notifications
- README style + docs/ scaffolding
- Security level
- Need a project-specific agent? (no / describe it)

#### 2. `workflow-research/SKILL.md` (~60 lines)
- Spawns 1-3 Explore agents based on scope
- Reads project CLAUDE.md for architecture context
- Searches codebase for patterns related to the requirement
- Optionally fetches external docs (WebSearch/WebFetch)
- Outputs structured research summary

#### 3. `workflow-plan/SKILL.md` (~80 lines)
- Takes requirement + research findings as input
- Spawns Plan agent to design implementation
- If project enables independent review: spawns a second read-only-researcher agent to critique
- Presents plan to user via EnterPlanMode → **pauses for approval**
- Stores approved plan in `.claude/plans/`

#### 4. `workflow-decompose/SKILL.md` (~50 lines)
- Takes approved plan
- Creates TaskList: one task per commit-sized unit
- Each task: description, files, test criteria, est. lines (respects project's max-lines setting)
- Displays swim lane if team mode is active

#### 5. `workflow-implement/SKILL.md` (~70 lines)
- Picks next incomplete task
- Creates worktree (if enabled in project config)
- If TDD: writes failing tests first, then implementation
- If test-after: writes implementation, then tests
- Runs project's formatter + linter (from CLAUDE.md config)
- Commits with conventional message
- Marks task complete

#### 6. `workflow-validate/SKILL.md` (~60 lines)
- Runs project's test command
- Checks coverage if configured
- Triggers code review per project's review gate setting
- Reports pass/fail with actionable failure details
- On failure: returns structured failure context for retry

#### 7. `workflow-cycle/SKILL.md` (~80 lines)
- Orchestrates: research → plan → ⏸ approve → decompose → implement(loop) → validate
- On validation failure: loops back to implement with failure context
- Stops after 3 failed retries (per global error recovery)
- Posts Slack summary on completion (if channel configured)

### B. Templates for Project Generation

Located at `~/.claude/skills/harness-bootstrap/templates/`.

#### `claude-md.template.md`
```
# {project_name}
{description}

## Build & Test
- Build: `{build_cmd}`
- Test: `{test_cmd}`
- Coverage: {coverage}%

## Code Style
- Formatter: `{format_cmd}`
- Linter: `{lint_cmd}`

## Testing
- Approach: {tdd|test-after}
- Required types: {test_types}
- Framework: {test_framework}

## Workflow
- Full cycle: `/cycle "requirement description"`
- Individual: `/research`, `/plan`, `/decompose`, `/implement`, `/validate`

## Commit Rules
- Format: Conventional Commits
- Max lines: {max_lines} (hard max: 400)
- Single logical change per commit

## Review Gate
- {review_gate_description}

## Planning
- Detail level: {plan_detail}
- Task granularity: {task_granularity}
- Independent plan review: {yes|no}

## Branching
- Worktrees: {yes|no}
- Merge strategy: {merge_strategy}
- Slack channel: {slack_channel}

## Security
- Level: {security_level}
- See @.claude/rules/security.md
```

#### `settings.template.json`
Generates project-level settings.json with:
- Language-appropriate permission rules (e.g., `Bash(npm *)` for TS, `Bash(bazel *)` for C++)
- Auto-format PostToolUse hook (language-specific formatter)
- Stop hook for uncommitted changes check

#### `rules/testing.template.md`
Agnostic core testing rules (TDD flow, coverage expectations, failure injection checklist).

#### `rules/security.template.md`
OWASP top 10 checklist + level-appropriate additions (crypto, audit logging for strict).

#### Language packs (`rules/{lang}.template.md`):
- **`cpp.md`**: Header guards, RAII, const-correctness, modern C++ idioms
- **`python.md`**: Type hints, `__all__` exports, dataclasses preference, async patterns
- **`typescript.md`**: Strict mode, no `any`, prefer `interface` over `type`, barrel exports
- **`java.md`**: Records preference, sealed classes, avoid checked exceptions

### C. Config Storage (`~/.claude/harness-configs/`)

Saved as YAML for readability:
```yaml
# python-cli.yaml
project_name: my-cli-tool
description: A CLI for managing deployments
language: python
build_tool: uv
test_framework: pytest
tdd: true
coverage: 80
test_types: [unit, integration, failure-injection]
plan_detail: medium
task_granularity: per-commit
max_lines: 200
review_gate: agent
worktrees: true
merge_strategy: rebase-ff
slack_channel: null
readme_style: standard
docs_scaffold: true
security_level: standard
custom_agent: null
```

### D. Compatibility Checker (embedded in bootstrap skill)

Reads `~/.claude/CLAUDE.md` and checks:

| Check | Global Rule | Action if Conflict |
|---|---|---|
| Build tool | Language→tool table | Flag: "Global says {X} for {lang}, you chose {Y}" |
| TDD | Default TDD | Flag: "Global enforces TDD. Override?" |
| Commit limit | 200 target, 400 max | Block if project > 400; flag if > 200 |
| Worktrees | Required | Flag: "Global requires worktrees" |
| Human review | Required for architecture | Flag if set to agent-only |
| Slack | Default #general | Warn if no channel specified |
| Formatters | Must run before commit | Auto-add if missing |
| Security | No hardcoded secrets etc. | Always include security rules |

---

## Implementation Plan

### Commit 1: Global workflow skills (research, plan, decompose)
**Files:**
- `~/.claude/skills/workflow-research/SKILL.md`
- `~/.claude/skills/workflow-plan/SKILL.md`
- `~/.claude/skills/workflow-decompose/SKILL.md`

### Commit 2: Global workflow skills (implement, validate, cycle)
**Files:**
- `~/.claude/skills/workflow-implement/SKILL.md`
- `~/.claude/skills/workflow-validate/SKILL.md`
- `~/.claude/skills/workflow-cycle/SKILL.md`

### Commit 3: Bootstrap skill + templates
**Files:**
- `~/.claude/skills/harness-bootstrap/SKILL.md`
- `~/.claude/skills/harness-bootstrap/templates/claude-md.template.md`
- `~/.claude/skills/harness-bootstrap/templates/settings.template.json`
- `~/.claude/skills/harness-bootstrap/templates/rules/testing.template.md`
- `~/.claude/skills/harness-bootstrap/templates/rules/security.template.md`

### Commit 4: Language packs
**Files:**
- `~/.claude/skills/harness-bootstrap/templates/rules/cpp.template.md`
- `~/.claude/skills/harness-bootstrap/templates/rules/python.template.md`
- `~/.claude/skills/harness-bootstrap/templates/rules/typescript.template.md`
- `~/.claude/skills/harness-bootstrap/templates/rules/java.template.md`

### Commit 5: Harness config directory + validation
**Files:**
- `~/.claude/harness-configs/.gitkeep`
**Actions:**
- Bootstrap a test project at `/tmp/test-harness/`
- Run `/harness-bootstrap` with Python CLI answers
- Run `/cycle "Add a CLI command that takes a name and prints a greeting"`
- Verify full flow completes successfully

---

## Verification Plan

1. **Skill loading**: Invoke `/harness-bootstrap` — verify all questions appear
2. **Config save**: Check `~/.claude/harness-configs/test-project.yaml` generated
3. **File generation**: Verify `.claude/CLAUDE.md`, `settings.json`, `rules/` created in target repo
4. **Compatibility check**: Intentionally answer "CMake" for C++ project — verify conflict flagged
5. **Workflow skills**: Run `/research "add greeting command"` — verify Explore agent spawns
6. **Plan approval**: Run `/plan` — verify it enters plan mode and pauses
7. **Full cycle**: Run `/cycle "hello world CLI"` — verify end-to-end flow
8. **Reuse**: Run `/harness-bootstrap` again in new repo — verify saved config offered

---

## Open Items for Review

- [ ] Review questionnaire categories — any questions missing or redundant?
- [ ] Review compatibility checker rules — any global rules not covered?
- [ ] Review workflow skill responsibilities — any overlap or gaps?
- [ ] Confirm language pack priorities (C++, Python, TS, Java) — add Go, Rust?
- [ ] Decide if `/cycle` should support partial runs (e.g., skip research if already done)
