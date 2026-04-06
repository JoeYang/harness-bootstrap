# Plugins Audit (Key Plugins Deep Dive)

The full plugin inventory is in `settings-audit.md`. This documents the 3 plugins most relevant to harness bootstrap and how our custom skills relate.

---

## superpowers (v5.0.7, by Jesse Vincent)

**What**: Core skills library -- 14 skills across the development lifecycle.

**Skill chain**: `brainstorming -> writing-plans -> executing-plans -> verification-before-completion -> finishing-a-development-branch`

| Skill | Phase | What it does |
|---|---|---|
| brainstorming | Design | Explores intent, requirements, alternatives before implementation |
| writing-plans | Design | Structured implementation plans from specs |
| executing-plans | Build | Executes plans with review checkpoints |
| test-driven-development | Build | Tests first, then implementation |
| subagent-driven-development | Build | Parallel task execution via sub-agents |
| dispatching-parallel-agents | Build | Routes independent tasks to parallel workers |
| systematic-debugging | Debug | Hypothesis-driven bug investigation |
| requesting-code-review | Review | Submits work for review before merge |
| receiving-code-review | Review | Evaluates review feedback with rigor |
| verification-before-completion | Review | Runs verification before claiming done |
| using-git-worktrees | Workflow | Creates isolated worktrees for feature work |
| finishing-a-development-branch | Workflow | Guides merge/PR/cleanup decisions |
| writing-skills | Meta | Creates and edits custom skills |
| using-superpowers | Meta | Bootstraps skill discovery at session start |

**Harness relevance**: Joe's CLAUDE.md duplicates several of these patterns (TDD, worktrees, branching). Rules should say WHEN to use a skill; the skill handles HOW.

---

## claude-code-setup (v1.0.0, by Anthropic)

**What**: Read-only codebase analyzer recommending Claude Code automations (hooks, skills, MCP servers, subagents).

**How**: Scans project files -> maps signals to automation types -> outputs 1-2 recommendations per category. Does NOT create or modify files.

**How our harness complements it**:
- Automation-recommender = upstream discovery ("you should add a format hook")
- `/harness-bootstrap` = downstream generation (creates the actual settings.json)
- `/harness-update` = incremental (detects new capabilities, applies them)

No conflict -- they chain naturally: recommend -> generate -> maintain.

---

## claude-md-management (v1.0.0, by Anthropic)

**What**: CLAUDE.md maintenance skills:
1. `claude-md-improver` -- Scores quality (A-F) against criteria (commands, architecture, gotchas, conciseness), proposes targeted updates with diffs. Can write after approval.
2. `/revise-claude-md` -- Listed in active skills but not present as a SKILL.md in the installed plugin directory. May be injected by the system separately.

**How our harness complements it**:
- Our harness generates initial CLAUDE.md for new projects from rules-library templates
- `claude-md-improver` runs after as a quality gate to optimize content
- `doc-sync` agent handles post-commit README + CLAUDE.md sync (different trigger, complementary)
