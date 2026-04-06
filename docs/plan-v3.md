# Harness Knowledge Base & Bootstrap Skill — Plan v3

> **Status**: Draft — pending review
> **Date**: 2026-04-06
> **Supersedes**: plan-v2.md
> **Changes from v2**: Dropped personal config generation, dropped template-filling, pre-fill from global config, scoped to project-only, addressed all v2 reviewer concerns

## Goal

This repo (`harness/`) is a **knowledge base and skill repository** for bootstrapping Claude Code **project environments**. It provides:

1. **`/harness-bootstrap`** — a skill that asks questions and generates `.claude/` project config
2. **Example configs** — well-crafted reference examples the skill uses when generating output
3. **Cookbooks** — practical hook and rules recipes with copy-paste patterns
4. **Reference docs** — how harness files work, scope precedence, compatibility

It does NOT:
- Generate personal/global config (`~/.claude/`) — that's mature and manually curated
- Replicate workflow orchestration — superpowers handles that
- Replace `claude-automation-recommender` — that plugin analyzes existing codebases; this bootstraps blank repos

### Relationship to Existing Tools

```
Blank new repo              Existing codebase          Ongoing development
     │                            │                          │
     ▼                            ▼                          ▼
/harness-bootstrap     /claude-automation-recommender    superpowers skills
  "Set up .claude/       "What hooks/skills should       (brainstorming,
   for this project"      I add to this project?"         writing-plans, TDD,
                                                          executing-plans...)
```

---

## Design Decisions

| Decision | Choice | Reason |
|---|---|---|
| Scope | Project-only (`.claude/`) | Personal config is mature; generating it risks destroying existing setup |
| Template approach | Example configs as LLM reference | Template-filling (`{placeholder}`) is unreliable LLM inference; examples are what LLMs are good at |
| Pre-fill from global | Skip predetermined questions, inform user | Reduces questionnaire from 16 to ~8-10 questions; respects existing global rules |
| Config format | JSON | Aligns with Claude Code native format; YAML indentation is fragile for LLMs |
| Conflict handling | Flag each conflict, user decides | Per-conflict resolution is safest |
| Monorepo | Not in v1 — add later | Keep scope tight for first version |
| Update mode | Not in v1 — initial setup only | Re-run and confirm overwrites is sufficient for now |
| Install method | Symlink + `HARNESS_DIR` env var | Portability with auto-sync |
| Research docs | Distilled into cookbooks | Raw research goes stale; practical recipes age slowly |

---

## Research-Backed Design Principles

Seven principles distilled from Anthropic docs, Trail of Bits, GitHub (2,500 repos), Cursor, Gemini CLI, Codex, and Windsurf:

1. **Commands first** — build/test/lint commands are the highest-value content in any config
2. **Keep it short** — Anthropic says <200 lines; every line competes for attention
3. **CLAUDE.md for advisory, hooks for enforcement** — suggestions vs guarantees
4. **Three-tier boundaries** — always do / ask first / never do (universal pattern)
5. **Progressive disclosure** — `@imports` for depth, root file stays concise
6. **Separate personal from team** — global `~/.claude/`, project `.claude/`, local `CLAUDE.local.md`
7. **Deny lists for credentials** — explicit deny on `~/.ssh/**`, `~/.aws/**`, etc.

---

## Repo Structure

```
harness/
├── README.md                           # What this repo is, when to use it vs alternatives
├── docs/
│   ├── plan-v1.md                      # Archived
│   ├── plan-v2.md                      # Archived
│   ├── plan-v3.md                      # This plan
│   ├── harness-anatomy.md              # Scopes, precedence, file roles, @import syntax
│   ├── hooks-cookbook.md                # Copy-paste hook recipes with explanations
│   ├── rules-cookbook.md                # Testing, security, language-specific rule patterns
│   └── compatibility-guide.md          # Global vs project interactions, conflict resolution
├── examples/                           # Reference examples the skill reads when generating
│   ├── python-cli/                     # Example: Python CLI project
│   │   ├── claude-md.example.md
│   │   ├── settings.example.json
│   │   └── rules/
│   │       ├── testing.example.md
│   │       └── security.example.md
│   ├── typescript-web/                 # Example: TypeScript web app
│   │   ├── claude-md.example.md
│   │   ├── settings.example.json
│   │   └── rules/
│   │       ├── testing.example.md
│   │       └── security.example.md
│   ├── cpp-bazel/                      # Example: C++ Bazel project
│   │   ├── claude-md.example.md
│   │   ├── settings.example.json
│   │   └── rules/
│   │       ├── testing.example.md
│   │       └── security.example.md
│   └── java-bazel/                     # Example: Java Bazel project
│       ├── claude-md.example.md
│       ├── settings.example.json
│       └── rules/
│           ├── testing.example.md
│           └── security.example.md
├── configs/                            # Saved bootstrap answers for reuse
│   └── .gitkeep
└── skill/
    └── SKILL.md                        # /harness-bootstrap
```

### Why examples/ instead of templates/

Templates with `{placeholders}` require deterministic string substitution — something LLMs do unreliably. Examples work because:
- The LLM reads a complete, working config for a similar project type
- It adapts the example to the user's specific answers
- This is what LLMs are actually good at (generation from examples, not mechanical substitution)
- Examples are also useful as standalone reference even without the skill

---

## The Bootstrap Skill (`/harness-bootstrap`)

### Flow

```
User runs /harness-bootstrap in a new project directory
         │
         ▼
┌──────────────────────────────────┐
│ 1. Read ~/.claude/CLAUDE.md      │ Extract global rules:
│    (global config)               │ TDD? worktrees? commit limits?
└────────────┬─────────────────────┘ build tool table? etc.
             ▼
┌──────────────────────────────────┐
│ 2. Check harness/configs/        │ "Found saved configs:
│    for reusable configs          │  python-cli, cpp-trading.
└────────────┬─────────────────────┘  Use one as starting point?"
             ▼
┌──────────────────────────────────┐
│ 3. Inform user of inherited      │ "From your global config:
│    settings from global config   │  TDD=required, worktrees=yes,
│                                  │  conventional commits, 200-line max"
└────────────┬─────────────────────┘
             ▼
┌──────────────────────────────────┐
│ 4. Ask remaining questions       │ Only questions where the answer
│    (2-3 rounds)                  │ isn't predetermined by global config
└────────────┬─────────────────────┘
             ▼
┌──────────────────────────────────┐
│ 5. Read matching example config  │ e.g., examples/python-cli/
│    from harness/examples/        │ as reference for generation
└────────────┬─────────────────────┘
             ▼
┌──────────────────────────────────┐
│ 6. Compatibility check           │ Flag if any answer contradicts
│    project vs global             │ global rules
└────────────┬─────────────────────┘
             ▼
┌──────────────────────────────────┐
│ 7. Generate .claude/ files       │ Adapted from example + answers
│    (with user confirmation)      │ Prompt before overwriting
└────────────┬─────────────────────┘
             ▼
┌──────────────────────────────────┐
│ 8. Save answers to               │ For reuse on future projects
│    harness/configs/{name}.json   │
└──────────────────────────────────┘
```

### Questionnaire (reduced — pre-filled from global)

**What's inherited (not asked):**
If `~/.claude/CLAUDE.md` exists, the skill reads it and extracts:
- TDD enforcement → skip "TDD or test-after?" question
- Worktree requirement → skip "Git worktrees?" question
- Commit format → skip "Commit message format?" question
- Commit line limits → skip "Max lines per commit?" question
- Review gate policy → may skip or narrow review question

The skill tells the user: "Inherited from your global config: TDD required, worktrees enabled, conventional commits, 200-line target (400 max). These apply to all projects."

**What's always asked (project-specific):**

**Round 1 — Project Identity & Commands:**

| # | Question | Type | Options |
|---|---|---|---|
| 1 | Project name + one-line description | Free text | (Other) |
| 2 | Primary language(s) | Single | C++, Java, Python, TypeScript |
| 3 | Build command | Single | Bazel, npm, uv, Auto-detect |
| 4 | Test command | Single | Auto-detect from build tool, Custom |

**Round 2 — Testing & Quality (only non-inherited questions):**

| # | Question | Type | Options |
|---|---|---|---|
| 5 | Test framework | Single | Auto-detect (gtest/pytest/jest/JUnit), Specify |
| 6 | Coverage target | Single | 80% (recommended), 90%, No target |
| 7 | Test types beyond unit | Multi | Integration, e2e, failure-injection, property-based |
| 8 | Security level | Single | Standard (OWASP), Strict (+crypto +audit), Minimal |

**Round 3 — Extras (conditional — only if not all inherited):**

| # | Question | Type | Options |
|---|---|---|---|
| 9 | Include language-specific rules? | Single | Yes, No |
| 10 | Slack notification channel | Single | #general, Custom, None |

If global config covers most settings, this may be only **2 rounds (8 questions)** instead of 4 rounds (16 questions). Faster, less room for context degradation.

### What Gets Generated

For a Python project with TDD, worktrees, conventional commits inherited from global:

```
project-root/
├── .claude/
│   ├── CLAUDE.md              # ~40-60 lines
│   ├── settings.json          # Permissions + hooks
│   └── rules/
│       ├── testing.md         # Path-scoped testing rules
│       ├── security.md        # Path-scoped security rules
│       └── python.md          # Language-specific (if opted in)
├── CLAUDE.local.md            # Empty placeholder (gitignored)
└── .gitignore                 # Updated with CLAUDE.local.md, .claude/settings.local.json
```

**Generated `.claude/CLAUDE.md`** follows the research-backed structure:

```markdown
# {Project Name}
{One-line description}

# Commands
- Build: `uv run python -m build`
- Test: `uv run pytest`
- Lint: `uv run ruff check`
- Format: `uv run ruff format`

# Architecture
- Source: `src/`
- Tests: `tests/`

# Testing
- Framework: pytest
- Coverage: 80%
- Required types: unit, integration, failure-injection
- See @.claude/rules/testing.md

# Boundaries
## Always Do
- Run `uv run pytest` before committing
- Run `uv run ruff format` before committing

## Ask First
- Changes to public API surface
- New dependencies
- Schema or data model changes

## Never Do
- Modify CI/CD without explicit approval
- Skip tests or disable test cases
- Hardcode secrets or credentials

# Security
- Level: standard
- See @.claude/rules/security.md
```

**Generated `.claude/settings.json`:**

```json
{
  "permissions": {
    "allow": [
      "Bash(uv *)",
      "Bash(python *)",
      "Bash(git *)",
      "Bash(ruff *)"
    ],
    "deny": [
      "Bash(rm -rf *)",
      "Read(.env*)"
    ]
  },
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [{
          "type": "command",
          "command": "FILE=$(echo '$CLAUDE_TOOL_INPUT' | jq -r '.file_path // empty'); [ -n \"$FILE\" ] && ruff format \"$FILE\" 2>/dev/null || true"
        }]
      }
    ]
  }
}
```

### Compatibility Check

Before writing files, the skill reads `~/.claude/CLAUDE.md` and flags contradictions:

| Project Answer | Global Rule | Action |
|---|---|---|
| Build: CMake | "C++ → Bazel" | Flag: "Global config says use Bazel for C++. Override for this project?" |
| Coverage: No target | "comprehensive coverage" | Flag: "Global config requires comprehensive test coverage." |
| Security: Minimal | "No hardcoded secrets" + OWASP | Flag: "Global config enforces OWASP. Minimal security may conflict." |

Non-contradictions are fine — the project config adds project-specific detail on top of global rules, it doesn't override them. Only flag actual conflicts.

### Config Save/Load

Saved as JSON to `harness/configs/{name}.json`:
```json
{
  "name": "my-python-cli",
  "created": "2026-04-06",
  "language": "python",
  "build_cmd": "uv run python -m build",
  "test_cmd": "uv run pytest",
  "lint_cmd": "uv run ruff check",
  "format_cmd": "uv run ruff format",
  "test_framework": "pytest",
  "coverage": 80,
  "test_types": ["unit", "integration", "failure-injection"],
  "security_level": "standard",
  "language_rules": true,
  "slack_channel": null
}
```

Note: inherited settings (TDD, worktrees, commit format) are NOT saved — they come from global config at runtime. This means configs remain valid even if global config changes.

### Portability

The skill needs to know where the harness repo lives to read examples and save configs. Solution:

Add to `~/.claude/settings.json`:
```json
{
  "env": {
    "HARNESS_DIR": "/home/joeyang/clawd/harness"
  }
}
```

The SKILL.md references `$HARNESS_DIR/examples/` and `$HARNESS_DIR/configs/`. If the env var is unset, the skill asks the user for the path.

---

## Reference Documentation (docs/)

### `harness-anatomy.md` (~100 lines)
What each harness file does, scope hierarchy, loading order, `@import` syntax. Distilled from Anthropic docs. Add `last-verified` date.

### `hooks-cookbook.md` (~120 lines)
Copy-paste recipes:
- Auto-format: prettier (TS), ruff (Python), clang-format (C++), google-java-format (Java)
- Branch protection: block edits on main (Trail of Bits pattern, exit code 2)
- File protection: block .env, lock files, CI config edits
- Stop check: warn on uncommitted changes
- Desktop notifications: notify-send + sound
- Auto-lint: run linter after write
- Post-compaction: re-inject critical context

### `rules-cookbook.md` (~120 lines)
- Testing: TDD flow, failure injection checklist, coverage gates
- Security: standard (OWASP) vs strict (+crypto, +audit logging)
- Language packs: C++ (RAII, const-correctness, modern idioms), Python (type hints, dataclasses), TS (strict mode, no any), Java (records, sealed classes)
- Path-scoped frontmatter examples
- Three-tier boundary pattern (always/ask/never)

### `compatibility-guide.md` (~80 lines)
- Scope precedence: managed > user > project > local (concatenation, not override)
- Permission merging: deny always wins across all scopes
- Hook merging: all hooks at all scopes fire (no override)
- Common conflict patterns and how to resolve them
- When to use CLAUDE.local.md vs ~/.claude/CLAUDE.md

---

## Example Configs (examples/)

Four complete, working examples — one per supported language. Each is a self-contained `.claude/` directory that could be dropped into a real project.

### `examples/python-cli/`
- Python + uv + pytest + ruff
- TDD, 80% coverage, failure injection
- Standard security

### `examples/typescript-web/`
- TypeScript + npm + vitest + prettier + eslint
- TDD, 80% coverage, e2e tests
- Standard security

### `examples/cpp-bazel/`
- C++ + Bazel + gtest + clang-format + clang-tidy
- TDD, failure injection, performance benchmarks
- Strict security

### `examples/java-bazel/`
- Java + Bazel + JUnit 5 + google-java-format + checkstyle
- TDD, 80% coverage, integration tests
- Standard security

---

## Implementation Plan

Line budget included per commit (target <200, hard max 400).

### Commit 1: Repo scaffold + README (~80 lines)
- `README.md` — what this repo is, when to use it vs claude-automation-recommender vs superpowers
- `configs/.gitkeep`
- Move existing plan docs are already in place

### Commit 2: Reference docs — anatomy + compatibility (~180 lines)
- `docs/harness-anatomy.md` (~100 lines)
- `docs/compatibility-guide.md` (~80 lines)

### Commit 3: Hooks cookbook (~120 lines)
- `docs/hooks-cookbook.md`

### Commit 4: Rules cookbook (~120 lines)
- `docs/rules-cookbook.md`

### Commit 5: Example — Python CLI (~90 lines)
- `examples/python-cli/claude-md.example.md`
- `examples/python-cli/settings.example.json`
- `examples/python-cli/rules/testing.example.md`
- `examples/python-cli/rules/security.example.md`

### Commit 6: Example — TypeScript web (~90 lines)
- `examples/typescript-web/claude-md.example.md`
- `examples/typescript-web/settings.example.json`
- `examples/typescript-web/rules/testing.example.md`
- `examples/typescript-web/rules/security.example.md`

### Commit 7: Example — C++ Bazel (~90 lines)
- `examples/cpp-bazel/claude-md.example.md`
- `examples/cpp-bazel/settings.example.json`
- `examples/cpp-bazel/rules/testing.example.md`
- `examples/cpp-bazel/rules/security.example.md`

### Commit 8: Example — Java Bazel (~90 lines)
- `examples/java-bazel/claude-md.example.md`
- `examples/java-bazel/settings.example.json`
- `examples/java-bazel/rules/testing.example.md`
- `examples/java-bazel/rules/security.example.md`

### Commit 9: Bootstrap skill (~150 lines)
- `skill/SKILL.md`
- Symlink: `ln -s $PWD/skill ~/.claude/skills/harness-bootstrap`
- Add `HARNESS_DIR` to `~/.claude/settings.json` env

### Commit 10: Validation
- Bootstrap a test project at `/tmp/test-harness/`
- Verify generated `.claude/` files are correct and complete
- Verify compatibility checker flags a deliberate conflict (e.g., CMake for C++)
- Verify config save/load roundtrip
- Verify generated project works with superpowers:writing-plans flow

---

## Addressing All v2 Review Concerns

| Concern | Resolution in v3 |
|---|---|
| C1: SKILL.md can't do template-filling reliably | Dropped templates. Example configs as LLM reference instead. |
| C2: claude-code-setup plugin overlap | Documented boundary in README. Different trigger: blank repo vs existing codebase. |
| C3: Personal config generation destroys existing setup | Dropped entirely. Project-only scope. |
| S4: Config portability (hardcoded paths) | `HARNESS_DIR` env var in settings.json. |
| S5: Templates are wrong abstraction | Replaced with example configs. |
| S6: Knowledge base goes stale | Kept cookbooks (age slowly). Dropped competitor comparisons. Added `last-verified` dates. |
| S7: Line count risk per commit | Pre-budgeted every commit. All under 200 lines. |
| 8: No upgrade path | Documented as future enhancement. v1 is initial setup only. |
| 9: Single language only | Questionnaire allows selecting primary language; multi-language is future. |
| 10: No monorepo support | Deferred to follow-up. Documented in README. |
| 11: Questions conflict with global config | Pre-fill from global. Skip inherited questions. Inform user. |
| 12: Symlink fragility | `HARNESS_DIR` env var + documented manual install as fallback. |

---

## Verification Plan

1. **Skill loads**: `/harness-bootstrap` invocable from any directory
2. **Global pre-fill**: Reads `~/.claude/CLAUDE.md`, correctly identifies inherited settings, informs user
3. **Reduced questions**: Only 2-3 rounds of questions (not 4) when global config covers most settings
4. **Example reading**: Skill reads matching example from `$HARNESS_DIR/examples/`
5. **Project generation**: `.claude/CLAUDE.md`, `settings.json`, `rules/` created correctly
6. **Conflict detection**: Answer "CMake" for C++ → conflict flagged against global Bazel rule
7. **No overwrite**: Run on repo with existing `.claude/` → prompts before overwriting
8. **Config roundtrip**: Save config → new project → reuse → same output
9. **Superpowers integration**: Generated config works with `superpowers:writing-plans` flow
10. **Boundary clarity**: README clearly explains when to use this vs claude-automation-recommender

---

## Future Enhancements (Not in v1)

- Monorepo subdirectory config generation
- Update/upgrade mode (re-run, apply diff)
- Multi-language project support
- Personal global config starter kit (for greenfield users)
- Go and Rust language examples
- Integration with claude-automation-recommender (run recommender after bootstrap)
