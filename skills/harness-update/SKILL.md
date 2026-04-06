---
name: harness-update
description: Update an existing project's .claude/ config with newly cataloged capabilities. Diffs installed vs available, shows what's new, applies selected updates. Use after adding new capabilities to the harness catalog.
disable-model-invocation: true
allowed-tools: Read, Glob, Grep, Bash(ls *), Write, Edit, AskUserQuestion
---

# /harness-update

Incrementally update a project's `.claude/` configuration by diffing installed capabilities against the catalog and applying new ones.

`HARNESS_DIR` is the absolute path to the harness repo. Read it from the `HARNESS_DIR` environment variable. If the variable is not set, use AskUserQuestion: "What is the absolute path to the harness-bootstrap repo?" All catalog/library paths below are relative to `$HARNESS_DIR`.

---

## Step 1: Read Current Config

Read the project's installed harness state:

- Read `.claude/CLAUDE.md` -- note sections and capabilities present
- Read `.claude/settings.json` -- note hooks and permissions
- List `.claude/rules/*.md` -- record each rule file name as an installed capability
- Read `.claude/settings.local.json` and look for the `harness` key:
  ```json
  {
    "harness": {
      "catalog_version": "1",
      "last_synced": "2026-04-06",
      "installed_capabilities": ["testing-rules:v1", "security-rules:v1", "python-rules:v1"]
    }
  }
  ```

If `settings.local.json` does not exist or has no `harness` key, this project was bootstrapped before version tracking. Infer installed capabilities from files: strip `.md` extension from each `rules/*.md` filename, append `-rules` if the stem has no hyphen (e.g., `testing` → `testing-rules`, `agent-teams` stays `agent-teams`), and tag as `:v0`. Treat catalog_version as `"0"`.

---

## Step 2: Read Catalog + Scan Project

Read `$HARNESS_DIR/docs/capability-catalog.md` to get the full list of available capabilities and the current catalog version from its header.

Scan the project for signals using the same detection as `/harness-bootstrap` Step 1:

- **Build tools**: `BUILD.bazel`, `package.json`, `pyproject.toml`, `pom.xml`, `Cargo.toml`
- **Test frameworks**: `*_test.*`, `test_*.py`, `conftest.py`, `*.test.ts`, `jest.config.*`, `vitest.config.*`
- **Formatters**: `.prettierrc*`, `.clang-format`, `ruff` in `pyproject.toml`
- **Linters**: `.eslintrc*`, `eslint.config.*`, `ruff` in `pyproject.toml`, `checkstyle.xml`
- **Frameworks** (from dependency files): `fastapi`, `flask`, `django`, `react`, `vue`, `express`, `grpc`, `.proto` files, `prisma`, `sqlalchemy`
- **Other signals**: `smoke_test_all.sh`, `Dockerfile`, `.github/workflows/`, `*.tf`

Build the list of all capabilities applicable to this project based on detected signals.

---

## Step 3: Diff Installed vs Available

Compare installed_capabilities against applicable capabilities. Categorize each:

- **New**: signal detected in project but capability not in installed list
- **Updated**: installed but catalog version is newer than last synced version (future -- flag as "update available" but do not auto-apply in v1)
- **Current**: installed and catalog version matches
- **Not applicable**: capability exists in catalog but no matching signal in this project

---

## Step 4: Present Update Menu

Show the diff summary using AskUserQuestion. Format:

```
Harness Update — {project directory name}
==========================================
Last synced: {last_synced or "never"}
Catalog version: {current catalog version}

New capabilities available:
  1. [ ] API design rules (FastAPI detected)
  2. [ ] Auto-lint hook (ruff config detected)
  3. [ ] gRPC rules (.proto files detected)

Already installed (current):
  [x] Testing rules
  [x] Security rules
  [x] Python rules

Not applicable (no signal detected):
  --- Exchange rules, Frontend rules
```

Ask: "Which new capabilities to install? (all / none / comma-separated list, e.g. 1,3)"

If nothing is new, report "All applicable capabilities are installed and current." and stop.

---

## Step 5: Apply Selected Updates

First, read all source material needed for the selected capabilities:
- Read `$HARNESS_DIR/docs/hooks-cookbook.md` if any selected capability is a hook
- Read relevant `$HARNESS_DIR/rules-library/*.md` files for selected rule capabilities

Then, for each selected capability:

1. **Rules file**: If source exists in `$HARNESS_DIR/rules-library/{capability}.md`, copy it to `.claude/rules/`. If not, generate from the catalog description and project context.
2. **Hook**: Use the pattern from hooks-cookbook.md (already read above). Add the hook entry to `.claude/settings.json` under the appropriate section (PreToolUse, PostToolUse, Stop).
3. **CLAUDE.md section**: Append to the appropriate section of `.claude/CLAUDE.md` (e.g., add a slack_channel line, add commands).

Before writing any file, show a preview of every change:
- For new files: show full content
- For modified files: show the diff (lines added/changed)

Use AskUserQuestion: "Apply these changes? (yes / edit first / cancel)"

If "edit first", ask which change to adjust, make the edit, and re-preview.

---

## Step 6: Update Version Marker

After applying changes, write or update `.claude/settings.local.json` with:

```json
{
  "harness": {
    "catalog_version": "{version from catalog header}",
    "last_synced": "{today's date, YYYY-MM-DD}",
    "installed_capabilities": ["{capability}:{catalog_version}", "...all installed -- previous + newly added"]
  }
}
```

Preserve any other keys already in `settings.local.json` -- only update the `harness` key.

Report completion: list each capability added and each file created or modified.
