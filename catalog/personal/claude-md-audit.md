# CLAUDE.md Section-by-Section Audit

Source: `~/.claude/CLAUDE.md` (224 lines)

## Build System
**Where**: lines 3-17 (15 lines) | **Candidate for**: Global only
**What**: Language-to-toolchain mapping (Bazel, uv, npm) with test commands.
**Why**: Without it, agent picks wrong build tools (e.g. Bazel on a simple TS project).
**Useful**: Every project. Compact, universal. **Skip**: Never.

## Project Planning
**Where**: lines 18-24 (7 lines) | **Candidate for**: Global only
**What**: Researcher agent review, feedback triage, ASCII diagrams, pros/cons.
**Why**: Agent implements first plan without considering alternatives. Triage prevents blind feedback application.
**Useful**: Non-trivial design work. **Skip**: Trivial scripts where planning overhead exceeds implementation.

## Testing
**Where**: lines 25-39 (15 lines) | **Candidate for**: Global only
**What**: TDD-first, failure injection tests (network, resource, race conditions), no skipping tests.
**Why**: Agent has strong happy-path bias. Without explicit failure injection rules, it only covers success cases.
**Useful**: Every project with tests. **Skip**: Prototypes or spikes with intentionally deferred coverage.

## Documentation
**Where**: lines 41-54 (14 lines) | **Candidate for**: Global only
**What**: README stays concise, detailed docs in docs/, docs updates must land in same commit as code changes. Documentation-only changes are exempt from the 200/400-line commit size limits.
**Why**: Without it, agent pads README with implementation details or forgets to update docs when code changes.
**Useful**: Always. **Skip**: Never.

## Scope
**Where**: lines 56-57 (2 lines) | **Candidate for**: Global only
**What**: Focus on current working directory only.
**Why**: Agent explores sibling directories, wastes context, makes cross-project assumptions.
**Useful**: Always. **Skip**: Never (monorepo scope should be explicit).

## Security
**Where**: lines 59-68 (10 lines) | **Candidate for**: Global only
**What**: No hardcoded secrets, input validation, OWASP top 10, least privilege, dependency security.
**Why**: Agent will hardcode API keys, skip validation, and introduce CVE-laden dependencies without this.
**Useful**: Always. **Skip**: Never.

## What the Agent Gets Wrong
**Where**: lines 70-76 (7 lines) | **Candidate for**: Global only
**What**: Guardrails for deprecated APIs, outdated deps, happy-path bias, inventing patterns, uncertainty.
**Why**: Observed failure modes from real sessions. Behavioral corrections, not project-specific.
**Useful**: Always. **Skip**: Never.

## Slack Notifications
**Where**: lines 78-81 (4 lines) | **Candidate for**: Both (global default + project channel override)
**What**: Post change summary to Slack after merging to main/master.
**Why**: Keeps team informed without manual effort. Channel configurable per project.
**Useful**: Team projects with Slack. **Skip**: Solo projects, learning exercises.

## Branching Strategy
**Where**: lines 83-92 (10 lines) | **Candidate for**: Both (core "never commit to main" stays global; worktree/skill details move to project level)
**What**: Never commit to main, use worktrees via superpowers skill, review before merge (human for architecture, agent for features).
**Why**: Without it, agent commits directly to main skipping review.
**Useful**: Every git project for the core rule. Worktree details only for projects with multi-branch workflows. **Skip**: Worktree specifics are noise for single-file scripts or throwaway repos.

## Exchange Smoke Test Gate
**Where**: lines 94-96 (3 lines) | **Candidate for**: Project-level only
**What**: Requires `./smoke_test_all.sh <exchange>` before completion.
**Why**: Validates full trading pipeline for exchange implementations.
**Useful**: Only 2-3 trading projects (exchange-connectivity, agency-trading).
**Skip**: Every other project. Pure noise for web apps, data pipelines, etc.

## Commit Rules
**Where**: lines 98-121 (24 lines) | **Candidate for**: Global only
**What**: 200-line target, 400-line max, atomic commits, conventional commit format. Three-step pre-commit: run formatters → run linters → self-review diff.
**Why**: Without size limits, agent produces 800-line monster commits mixing features, refactors, and migrations.
**Useful**: Always. **Skip**: Never.

## Model Selection
**Where**: lines 123-141 (19 lines) | **Candidate for**: Both (model routing table stays global; "human review mandatory" block moves to project level for production/architectural changes)
**What**: Task-to-model routing (Opus for design, Sonnet for implementation, Haiku for trivial) plus mandatory human review gate for architecture decisions.
**Why**: Controls cost and latency. Without it, every task uses the most expensive model.
**Useful**: Every project for model routing. **Skip**: The "human review mandatory" block (lines 137-141) is project-level — skip for learning/spike projects where architectural gates add friction.

## Error Recovery
**Where**: lines 142-148 (7 lines) | **Candidate for**: Global only
**What**: Stop after 3 failed attempts, escalate, test after every change, no workarounds.
**Why**: Without it, agent loops endlessly or silently works around problems by disabling tests.
**Useful**: Always. **Skip**: Never.

## Context Management
**Where**: lines 149-153 (5 lines) | **Candidate for**: Global only
**What**: Run /compact between tasks, start fresh, don't /clear when stuck.
**Why**: Stale context causes agent to reference outdated file contents in next task.
**Useful**: Always. **Skip**: Never.

## Agent Team Orchestration
**Where**: lines 155-215 (61 lines) | **Candidate for**: Project-level only
**What**: Lead/teammate roles, task protocol, worktree discipline, merge strategy, swim lanes.
**Why**: Without it, lead implements directly, teammates write to main, merge conflicts go unresolved.
**Useful**: Multi-agent team sessions on complex projects needing parallel work.
**Skip**: Single-agent sessions, small projects, quick fixes. This is 61 lines (27% of the file) and loads in every session even though most sessions are single-agent. Biggest candidate for extraction.

## Session Memory
**Where**: lines 217-224 (7 lines) | **Candidate for**: Global only
**What**: Read MEMORY.md at session start, update at session end.
**Why**: Without it, each session starts from scratch with no knowledge of prior decisions or blockers.
**Useful**: Multi-session projects. **Skip**: One-shot tasks.

---

## Summary

| Category | Lines | Candidate |
|---|---|---|
| Global only (11 sections) | ~138 | Stays in ~/.claude/CLAUDE.md |
| Project-level only (2 sections) | 64 | Exchange Smoke Test + Agent Team Orchestration |
| Both (3 sections) | ~33 | Slack Notifications, Branching Strategy, Model Selection |

Key insight: Agent Team Orchestration (61 lines) and Exchange Smoke Test (3 lines) together account for 64 lines (29% of the file) but only apply to 2-3 trading projects. Extracting them to project-level configs would slim the global file by nearly a third.
