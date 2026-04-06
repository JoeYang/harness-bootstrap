# Harness Knowledge Base & Bootstrap Skill — Plan v2

> **Status**: Draft — pending review
> **Date**: 2026-04-06
> **Supersedes**: plan-v1.md (over-engineered — tried to replicate superpowers workflow)
> **Research**: Anthropic docs, Trail of Bits, GitHub AGENTS.md (2,500 repos), Cursor, Gemini CLI, Codex, Windsurf

## Goal

This repo (`harness/`) is a **knowledge base and skill repository** for bootstrapping Claude Code environments. It accumulates harness knowledge (docs, templates, cookbooks) and provides one main skill — `/harness-bootstrap` — that interactively configures either:

- **Personal environment** (`~/.claude/`) — global standards, permissions, hooks, agents
- **Project environment** (`.claude/`) — project CLAUDE.md, settings, rules, hooks

It does NOT replicate workflow orchestration (superpowers handles that). The generated config references existing tools, doesn't replace them.

---

## Design Decisions (User-Confirmed)

| Decision | Choice |
|---|---|
| UX style | Interactive first run, saves config to `harness/configs/` for reuse |
| Skill scope | One global skill (`/harness-bootstrap`) + knowledge base in repo |
| Workflow | No workflow skills — superpowers handles orchestration |
| Rules | Agnostic core + optional language-specific packs |
| Agents | Use global agents; generate project-specific only if user identifies a gap |
| Config format | JSON (not YAML — per reviewer feedback) |
| Conflict handling | Flag each conflict, user decides per-conflict |
| Research docs | Distilled into polished reference articles (not raw agent output) |
| Install method | Symlink from `harness/skill/` → `~/.claude/skills/harness-bootstrap/` |

---

## Research-Backed Design Principles

From analyzing how Anthropic, Trail of Bits, GitHub (2,500 repos), Cursor, Gemini CLI, Codex, and Windsurf approach this:

### 1. Commands First
Every tool's best practice leads with build/test/lint commands. This is the single highest-value content in any config file. The bootstrap questionnaire should capture these before anything else.

### 2. Keep It Short
Anthropic: <200 lines. HumanLayer: <60 lines. Trail of Bits fits their entire global config in one template. The system prompt already has ~50 built-in instructions — every line you add competes for attention.

### 3. CLAUDE.md for Advisory, Hooks for Enforcement
CLAUDE.md instructions are suggestions. Hooks are deterministic and guaranteed. Production setups (Trail of Bits, ChrisWiles) use CLAUDE.md for conventions and hooks for things that must never be violated.

### 4. Three-Tier Boundary System
From GitHub's analysis of 2,500+ repos, the most effective instruction files use:
- **Always do**: build commands, test commands, formatting
- **Ask first**: architecture changes, schema changes, dependency upgrades
- **Never do**: push to main, rm -rf, skip hooks, hardcode secrets

### 5. Progressive Disclosure via @imports
Keep the root file concise. Use `@docs/testing.md`, `@docs/security.md` imports for depth. This is how Trail of Bits, Joe Cotellese, and HumanLayer all structure their configs.

### 6. Separate Personal from Team
Every tool solves this the same way:
- Global personal: `~/.claude/CLAUDE.md` (your standards for all projects)
- Project team: `./CLAUDE.md` (committed to git, shared)
- Local personal: `./CLAUDE.local.md` (gitignored, your overrides for this project)

### 7. Deny Lists for Credentials
Every production setup explicitly denies credential paths: `~/.ssh/**`, `~/.aws/**`, `~/.kube/**`, crypto wallets. Trail of Bits has the most comprehensive deny list.

---

## Repo Structure

```
harness/
├── README.md                           # What this repo is, how to use it
├── docs/
│   ├── plan-v1.md                      # Archived original plan
│   ├── plan-v2.md                      # This plan
│   ├── harness-anatomy.md              # What each harness file does, scopes, precedence
│   ├── hooks-cookbook.md                # Production hook recipes (auto-format, protect, lint)
│   ├── rules-cookbook.md                # Effective .claude/rules/ patterns
│   ├── compatibility-guide.md          # Global vs project config interactions
│   └── research/
│       ├── anthropic-setup.md          # How Anthropic recommends setup (distilled)
│       ├── industry-comparison.md      # Claude vs Copilot vs Cursor vs Gemini vs Codex vs Windsurf (distilled)
│       └── real-world-examples.md      # Trail of Bits, ChrisWiles, etc. (distilled)
├── templates/                          # Fillable templates for generation
│   ├── personal/
│   │   ├── claude-md.template.md       # ~/.claude/CLAUDE.md template
│   │   ├── settings.template.json      # ~/.claude/settings.json template
│   │   └── deny-list.template.json     # Credential path deny rules (Trail of Bits pattern)
│   ├── project/
│   │   ├── claude-md.template.md       # .claude/CLAUDE.md template
│   │   ├── settings.template.json      # .claude/settings.json template
│   │   └── gitignore-additions.txt     # Lines to add to .gitignore
│   ├── rules/
│   │   ├── testing.template.md         # Agnostic testing rules
│   │   ├── security.template.md        # Security rules (standard + strict tiers)
│   │   ├── cpp.template.md             # C++ language pack
│   │   ├── python.template.md          # Python language pack
│   │   ├── typescript.template.md      # TypeScript language pack
│   │   └── java.template.md            # Java language pack
│   └── hooks/
│       ├── auto-format.template.json   # PostToolUse formatter (per-language variants)
│       ├── branch-protection.template.json  # PreToolUse: block edits on main
│       └── stop-check.template.json    # Stop: warn on uncommitted changes
├── configs/                            # Saved bootstrap configs for reuse
│   └── .gitkeep
└── skill/
    └── SKILL.md                        # /harness-bootstrap — the main skill
```

---

## The Bootstrap Skill (`/harness-bootstrap`)

### How It Works

```
User runs /harness-bootstrap
         │
         ▼
┌─────────────────────────────┐
│ 1. Check for saved configs  │──→ Offer to reuse existing config
│    in harness/configs/      │
└────────────┬────────────────┘
             ▼
┌─────────────────────────────┐
│ 2. Ask: Personal or Project │
│    or Both?                 │
└────────────┬────────────────┘
             ▼
┌─────────────────────────────┐
│ 3. Run questionnaire        │
│    (4 rounds of questions)  │
└────────────┬────────────────┘
             ▼
┌─────────────────────────────┐
│ 4. Compatibility check      │──→ Flag conflicts with existing config
│    against existing config  │
└────────────┬────────────────┘
             ▼
┌─────────────────────────────┐
│ 5. Generate files from      │
│    templates                │
└────────────┬────────────────┘
             ▼
┌─────────────────────────────┐
│ 6. Save config to           │
│    harness/configs/         │
└─────────────────────────────┘
```

### Questionnaire (4 rounds, up to 4 questions each)

Structured around the 6 areas GitHub found essential across 2,500 repos: **commands, testing, structure, style, workflow, boundaries**.

**Round 1 — Scope & Commands** (the highest-value information):

| # | Question | Type | Options |
|---|---|---|---|
| 1 | Personal env, project env, or both? | Single | Personal / Project / Both |
| 2 | Primary language | Single | C++, Java, Python, TypeScript |
| 3 | Build command | Single | Bazel, npm, uv, Auto-detect |
| 4 | Test command | Single | `bazel test //...`, `npm test`, `uv run pytest`, Custom |

**Round 2 — Testing & Quality:**

| # | Question | Type | Options |
|---|---|---|---|
| 5 | TDD or test-after? | Single | TDD (recommended), Test-after |
| 6 | Coverage target | Single | 80%, 90%, No target |
| 7 | Test types | Multi | Unit, integration, e2e, failure-injection |
| 8 | Code review gate | Single | Human+agent hybrid, Agent only, Human only |

**Round 3 — Workflow & Style:**

| # | Question | Type | Options |
|---|---|---|---|
| 9 | Git worktrees for isolation? | Single | Yes (recommended), No |
| 10 | Merge strategy | Single | Rebase+FF, Squash, Standard merge |
| 11 | Max lines per commit | Single | 200 (recommended), 300, 400 |
| 12 | Commit message format | Single | Conventional Commits, Freeform |

**Round 4 — Boundaries & Extras:**

| # | Question | Type | Options |
|---|---|---|---|
| 13 | Security level | Single | Standard (OWASP), Strict (+crypto +audit), Minimal |
| 14 | Include language-specific rules? | Single | Yes, No |
| 15 | Include credential deny list? (personal only) | Single | Full (Trail of Bits), Basic, None |
| 16 | Slack notification channel | Single | #general, Custom, None |

### What Gets Generated

**For personal env (`~/.claude/`):**

| File | Content | Source template |
|---|---|---|
| `CLAUDE.md` | Global coding standards, commit rules, testing approach, three-tier boundaries | `templates/personal/claude-md.template.md` |
| `settings.json` | Permissions (allow/deny), hooks (notification, stop-check), env vars | `templates/personal/settings.template.json` |
| Deny rules | Credential path deny list merged into settings.json | `templates/personal/deny-list.template.json` |

Generated `~/.claude/CLAUDE.md` structure (following research-backed pattern):
```markdown
# Commands
- Build: `{build_cmd}`
- Test: `{test_cmd}`
- Lint: `{lint_cmd}`
- Format: `{format_cmd}`

# Testing
- Approach: {TDD / test-after}
- Coverage: {target}%
- Required types: {types}

# Code Style
- {language-specific rules via @import if opted in}

# Workflow
- Commits: {Conventional Commits}, max {N} lines
- Branching: {worktree strategy}
- Merge: {strategy}
- Review: {gate type}

# Boundaries
## Always Do
- Run tests before committing
- Run formatters before committing
- Use feature branches

## Ask First
- Architecture changes
- Schema changes
- Dependency upgrades

## Never Do
- Push directly to main/master
- Skip pre-commit hooks (--no-verify)
- Hardcode secrets
- Use rm -rf without confirmation
```

**For project env (`.claude/`):**

| File | Content | Source template |
|---|---|---|
| `CLAUDE.md` | Project name, build/test commands, architecture notes, workflow references | `templates/project/claude-md.template.md` |
| `settings.json` | Project-specific permissions, auto-format hooks | `templates/project/settings.template.json` |
| `rules/testing.md` | Testing rules with path scope | `templates/rules/testing.template.md` |
| `rules/security.md` | Security checklist with path scope | `templates/rules/security.template.md` |
| `rules/{lang}.md` | Language-specific rules (if opted in) | `templates/rules/{lang}.template.md` |
| `.gitignore` additions | `CLAUDE.local.md`, `settings.local.json` | `templates/project/gitignore-additions.txt` |

Generated `.claude/CLAUDE.md` structure:
```markdown
# {Project Name}
{One-line description}

# Commands
- Build: `{build_cmd}`
- Test: `{test_cmd}`
- Lint: `{lint_cmd}`
- Format: `{format_cmd}`

# Architecture
- See @docs/architecture.md for details
- {brief directory layout}

# Testing
- Approach: {TDD / test-after}
- Framework: {framework}
- Coverage: {target}%
- See @.claude/rules/testing.md

# Boundaries
## Always Do
- Run `{test_cmd}` before committing
- Run `{format_cmd}` before committing

## Ask First
- Changes to public API surface
- New dependencies

## Never Do
- Modify CI/CD without explicit approval
- Skip tests

# Security
- See @.claude/rules/security.md
```

### Compatibility Check

Before writing files, the skill checks for conflicts:

**If generating personal config:**
- Read existing `~/.claude/CLAUDE.md` → warn if it will be overwritten
- Read existing `~/.claude/settings.json` → merge permissions/hooks, don't clobber
- Show diff of what will change → user confirms

**If generating project config:**
- Read existing `.claude/` → warn if files will be overwritten
- Read `~/.claude/CLAUDE.md` (global) → flag if project answers contradict global rules:
  - Different build tool for the language
  - TDD disabled when global enforces it
  - Commit limit exceeds global maximum
  - Worktrees disabled when global requires them
- Present each conflict → user decides per-conflict

### Config Save/Load

Saved as JSON to `harness/configs/{name}.json`:
```json
{
  "name": "python-cli",
  "created": "2026-04-06",
  "scope": "project",
  "language": "python",
  "build_cmd": "uv run python -m build",
  "test_cmd": "uv run pytest",
  "tdd": true,
  "coverage": 80,
  "test_types": ["unit", "integration", "failure-injection"],
  "review_gate": "agent",
  "worktrees": true,
  "merge_strategy": "rebase-ff",
  "max_lines": 200,
  "commit_format": "conventional",
  "security_level": "standard",
  "language_rules": true,
  "deny_list": "basic",
  "slack_channel": null
}
```

On next run, skill offers: "Found saved configs: python-cli, cpp-trading. Use one as a starting point?"

---

## Reference Documentation (docs/)

Distilled from research into polished, maintainable reference articles.

### `harness-anatomy.md`
- What each file does: CLAUDE.md, settings.json, rules/, skills/, agents/
- Scope hierarchy: managed > user > project > local
- How files are loaded (walk up directory tree, concatenate)
- CLAUDE.md vs CLAUDE.local.md vs .claude/CLAUDE.md
- The `@import` syntax for progressive disclosure

### `hooks-cookbook.md`
Copy-paste hook recipes with explanations:
- Auto-format on edit (per language: prettier, black, clang-format, rustfmt)
- Block edits on main branch (Trail of Bits pattern)
- Protect sensitive files from edits
- Auto-run linter after write
- Uncommitted changes warning on stop
- Desktop notifications (notify-send + sound)
- Context re-injection after compaction
- Auto-run associated test file after source edit

### `rules-cookbook.md`
- Testing rules: TDD flow, failure injection checklist, coverage gates
- Security rules: OWASP top 10, crypto policy, audit logging
- Language packs: C++ (RAII, const-correctness), Python (type hints, dataclasses), TS (strict mode), Java (records, sealed)
- Path-scoped frontmatter examples
- The three-tier boundary pattern (always/ask/never)

### `compatibility-guide.md`
- Global vs project precedence (concatenation, not override)
- Permission merging (deny always wins)
- Hook merging (all hooks at all scopes fire)
- Common conflict patterns and resolutions
- When to use CLAUDE.local.md vs ~/.claude/CLAUDE.md

### `research/` (distilled reference material)
- `anthropic-setup.md` — Official Anthropic recommendations
- `industry-comparison.md` — Claude vs Copilot vs Cursor vs Gemini vs Codex vs Windsurf
- `real-world-examples.md` — Trail of Bits, ChrisWiles, etc.

---

## Implementation Plan

### Commit 1: Repo structure + README
- `harness/README.md`
- `docs/plan-v1.md` (already exists)
- `docs/plan-v2.md` (this plan)

### Commit 2: Research docs (distilled)
- `docs/research/anthropic-setup.md`
- `docs/research/industry-comparison.md`
- `docs/research/real-world-examples.md`

### Commit 3: Reference docs (harness anatomy + compatibility)
- `docs/harness-anatomy.md`
- `docs/compatibility-guide.md`

### Commit 4: Cookbooks (hooks + rules)
- `docs/hooks-cookbook.md`
- `docs/rules-cookbook.md`

### Commit 5: Templates (personal + project)
- `templates/personal/claude-md.template.md`
- `templates/personal/settings.template.json`
- `templates/personal/deny-list.template.json`
- `templates/project/claude-md.template.md`
- `templates/project/settings.template.json`
- `templates/project/gitignore-additions.txt`

### Commit 6: Templates (rules + hooks)
- `templates/rules/testing.template.md`
- `templates/rules/security.template.md`
- `templates/hooks/auto-format.template.json`
- `templates/hooks/branch-protection.template.json`
- `templates/hooks/stop-check.template.json`

### Commit 7: Language packs
- `templates/rules/cpp.template.md`
- `templates/rules/python.template.md`
- `templates/rules/typescript.template.md`
- `templates/rules/java.template.md`

### Commit 8: Bootstrap skill + install
- `skill/SKILL.md`
- `configs/.gitkeep`
- Symlink: `skill/` → `~/.claude/skills/harness-bootstrap/`

### Commit 9: Validation
- Bootstrap a test project at `/tmp/test-harness/`
- Verify generated files
- Verify compatibility checker
- Verify config save/load

---

## Verification Plan

1. **Skill loads**: `/harness-bootstrap` invocable from any directory
2. **Questions render**: All 4 rounds of AskUserQuestion work correctly
3. **Personal generation**: `~/.claude/CLAUDE.md` changes proposed with diff preview
4. **Project generation**: `.claude/CLAUDE.md`, `settings.json`, `rules/` created correctly
5. **Conflict detection**: Answer "CMake" for C++ → conflict flagged against global Bazel rule
6. **Config roundtrip**: Save config → new run → offer to reuse → generates same output
7. **Existing config safety**: Run on repo with existing `.claude/` → prompts before overwrite
8. **Generated config works**: Created project successfully uses superpowers:writing-plans flow
