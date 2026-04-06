---
name: harness-bootstrap
description: Bootstrap a project's .claude/ configuration. Scans project, reads capability catalog, asks adaptive questions, generates CLAUDE.md + settings.json + rules. Use when setting up Claude Code for a new or existing project.
disable-model-invocation: true
allowed-tools: Read, Glob, Grep, Bash(ls *), Write, Edit, AskUserQuestion
---

# /harness-bootstrap

Bootstrap a project's `.claude/` configuration by scanning the project, matching against the capability catalog, and generating tailored config files.

`HARNESS_DIR` is the absolute path to the harness repo. Read it from the `HARNESS_DIR` environment variable. If the variable is not set, use AskUserQuestion: "What is the absolute path to the harness-bootstrap repo?" All paths below are relative to `$HARNESS_DIR`.

---

## Step 1: Scan Project

Detect the project's stack using Glob and Grep. Check for:

**Build tools**: `BUILD.bazel`, `package.json`, `pyproject.toml`, `pom.xml`, `Cargo.toml`

**Test frameworks**: `*_test.*`, `test_*.py`, `conftest.py`, `*.test.ts`, `*Test.java`, `jest.config.*`, `vitest.config.*`

**Formatters**: `.prettierrc*`, `.clang-format`, `ruff` in `pyproject.toml`

**Linters**: `.eslintrc*`, `eslint.config.*`, `ruff` in `pyproject.toml`, `checkstyle.xml`

**Frameworks** (from dependency files):
- Python: `fastapi`, `flask`, `django`, `sqlalchemy`, `alembic` in `pyproject.toml`
- JS/TS: `react`, `vue`, `svelte`, `express`, `next` in `package.json`
- Any: `grpc`, `.proto` files, `ws`/`socket.io`, `prisma`, `typeorm`

**Other signals**: `smoke_test_all.sh`, `Dockerfile`, `.github/workflows/`, `*.tf`, `.proto`

Record findings as: language, build_tool, test_framework, formatter, linter, frameworks[], domain_signals[].

---

## Step 2: Read Capability Catalog

Read `$HARNESS_DIR/docs/capability-catalog.md`. For each detected signal, find the matching capability row and assign its tier:
- **Tier 1 (auto-included)**: Add directly, no question needed
- **Tier 2 (confirm/adjust)**: Queue for user confirmation
- **Tier 3 (custom)**: Signals with no catalog match -- note for later

Build three lists: `auto_included[]`, `confirm_adjust[]`, `unmatched_signals[]`.

---

## Step 3: Read Global Config

Read `~/.claude/CLAUDE.md` if it exists. Extract inherited rules (TDD, commit format, security baseline, branching strategy) that overlap with catalog items. Record as `inherited_rules[]` -- skip questions for these, do NOT duplicate at project level.

---

## Step 4: Check Saved Configs

List files in `$HARNESS_DIR/configs/*.json`. If the directory is empty or no `.json` files exist, skip this step entirely. If configs exist, show the list (name, language, description) and use AskUserQuestion: "Reuse an existing config as starting point? Enter name, or 'no' for fresh." If reusing, read that JSON and pre-fill answers for Steps 5-6.

---

## Step 5: Present Discovery and Ask Questions

Display the analysis summary to the user:
```
Project Analysis
================
Language:    {language}
Build:       {build_tool}
Test:        {test_framework}
Formatter:   {formatter}
Linter:      {linter}
Frameworks:  {frameworks}
Signals:     {domain_signals}

Auto-included (Tier 1): {list}
Inherited from global:  {list}
```

Ask questions in two rounds. Within each round, batch all questions into a single AskUserQuestion call. Do not call AskUserQuestion once per question.

**Round 1 -- Identity and detections:**
- "Project name and one-line description?"
- Confirm/correct each Tier 2 detection (build cmd, test cmd, format cmd, lint cmd)
- "Does this project use agent teams?" (not auto-detectable)
- "Is this a trading/exchange project?" (unless `smoke_test_all.sh` detected)

**Round 2 -- Preferences:**
- "Test coverage target? (80% / 90% / all-paths / skip)" -- skip if global covers it. "skip" maps to null in saved config.
- "Security level: standard (OWASP) / strict (+ crypto, audit) / minimal?"
- "Include {language} language rules?" (for detected language)
- "Slack notification channel? (e.g., #my-project, or skip)"
- For each detected domain capability (API, DB, frontend, Docker, etc.): "Include {capability} rules?"

For unmatched signals (Tier 3), ask: "Detected {signal} with no pre-built rules. Generate custom rules for it?"

Only ask questions where the answer cannot be auto-detected or inherited from global config.

---

## Step 6: Read Matching Example

Based on detected language, read the closest example from `$HARNESS_DIR/examples/`:

| Language | Example Dir |
|---|---|
| Python | `examples/python-cli/` |
| TypeScript/JavaScript | `examples/typescript-web/` |
| C++ | `examples/cpp-bazel/` |
| Java | `examples/java-bazel/` |

Read `claude-md.example.md`, `settings.example.json`, and any files under `rules/`.

If no matching example exists for the detected language, use `examples/python-cli/` as a structural reference and note in the output that no language-specific example was available.

Use these as the style/structure reference. Generated files must match the example's tone -- do not invent new structural patterns.

---

## Step 7: Generate .claude/ Files

Generate three categories of files, adapting the example to user answers:

**CLAUDE.md**: Project header, commands (build/test/format/lint), architecture notes, boundaries (always do / ask first / never do), slack channel. Do not duplicate content that lives in `rules/` files.

**settings.json**: `permissions.allow` (language-appropriate), `permissions.deny` (credentials), hooks (auto-format, auto-lint, branch protection, stop check) based on detected tools and user answers.

**rules/ files**: Two sources — copy from rules-library or generate from context:

Copy from `$HARNESS_DIR/rules-library/` (these files exist):
- `agent-teams.md` -- if user opted in
- `exchange.md` + `trading-latency.md` -- if trading project
- `api-design.md`, `database.md`, `frontend.md` -- if user opted in

Generate from example + catalog (these do NOT have library files):
- `rules/testing.md` -- adapted to detected framework and coverage target
- `rules/security.md` -- at the chosen security level
- `rules/{language}.md` -- if user opted in
- Docker, gRPC, WebSocket, IaC rules -- generate from project context (Tier 3)

**Before writing**, show a full preview of every file. If `.claude/` already exists, list which files will be overwritten. Use AskUserQuestion: "Write these files? (yes / edit first / cancel)". If "edit first", ask which file to adjust, make changes, and re-preview.

---

## Step 8: Save Config

Save the bootstrap answers to `$HARNESS_DIR/configs/{project-name}.json`.

Include: name, description, language, build_cmd, test_cmd, format_cmd, lint_cmd, coverage_target, security_level, slack_channel, agent_teams (bool), exchange_project (bool), included_rules[], custom_rules[], created (date).

Do NOT save inherited settings -- those come from global config at runtime.

Report completion: list all files written and their line counts.
