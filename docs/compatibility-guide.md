# Compatibility Guide

How global and project configs interact, merge, and conflict.

---

## CLAUDE.md Layering

CLAUDE.md files **concatenate** -- they do not override. The harness collects every CLAUDE.md from managed, global, project, and local scopes, then feeds them all into context as one block. Parent directories are included (walk up the tree).

Practical effect: if global says "use TDD" and project says "use pytest", the agent sees both instructions. They stack, not replace.

`CLAUDE.local.md` is read last and appears at the end of the concatenated context. Since LLMs weight recent context more heavily, local instructions have the strongest effective influence.

---

## Permission Merging

`permissions.deny` always wins across all scopes. There is no way to override a deny.

- Global denies `rm -rf *` -> project cannot allow it
- Project denies `Read(.env*)` -> local settings cannot allow it
- `permissions.allow` is additive: global allows + project allows = union of both

This is a security invariant, not a convention. The harness enforces it.

---

## Hook Merging

All hooks at all scopes fire. A project hook does NOT replace a global hook -- both run.

- Global has a `Notification` hook (desktop chime)
- Project adds a `Notification` hook (Slack ping)
- Result: both fire on every notification

Hook execution order: managed hooks fire first, then project, then user (global) last. This is independent of settings merge precedence. For PreToolUse hooks, exit code 2 blocks the action. Stop hooks fire after the agent has already finished — a non-zero exit signals a hook error, not an agent block.

---

## Common Conflict Patterns

### Contradictory instructions

Global says "use Bazel for C++" but project says "use CMake".

**What happens**: The agent sees both instructions. It may follow either one -- the result is unpredictable. Whichever instruction appears later in context (project) has a slight edge, but this is not reliable.

**Fix**: The project CLAUDE.md should not contradict global rules. If the project genuinely needs CMake, the global rule should be scoped more narrowly ("Bazel for C++ unless project specifies otherwise") or the project should use `CLAUDE.local.md` with an explicit override note.

### Redundant instructions

Global requires TDD. Project CLAUDE.md also says "use TDD".

**What happens**: The agent sees the same instruction twice. Harmless but wasteful -- burns context tokens and adds noise.

**Fix**: Project CLAUDE.md should reference or extend global rules, not repeat them. Write "see global TDD rules; additionally, use pytest fixtures for..." instead of restating the TDD policy.

### Ambiguous layering

Global says "run tests after every change". Project says "batch integration tests at the end". The agent sees both and must reconcile them -- it may follow one and ignore the other.

**Fix**: Be explicit about which rule governs which scope. "Unit tests: after every change. Integration tests: batch at end."

---

## When to Use Each Scope

| Scope | Use for | Examples |
|---|---|---|
| `~/.claude/CLAUDE.md` | Personal coding standards that apply everywhere | TDD preference, commit style, model selection |
| `./CLAUDE.md` | Project-specific commands, architecture, team norms | Build/test/lint commands, module boundaries |
| `./CLAUDE.local.md` | Personal overrides for this project (gitignored) | "I'm working on module X, focus there" |
| `.claude/rules/` | File-path-scoped rules loaded conditionally | "API files must use zod validation" |
| `.claude/settings.json` | Team permissions and hooks (committed) | Allow list, format-on-save hook |
| `.claude/settings.local.json` | Personal permission overrides (gitignored) | Extra MCP servers, stale one-off permissions |
| `.claude/skills/` | Project-specific workflows and domain knowledge | Deploy script, component generator |
| `.claude/agents/` | Project-specific specialist sub-agents | Domain-expert reviewer, test generator |
| `.mcp.json` | Project MCP servers (committed, team-shared) | Database, Sentry, project-specific tools |

---

## Anti-patterns

**Don't repeat global rules in project CLAUDE.md.** Both files concatenate -- the agent sees the rule twice, wasting context. Reference instead of repeat.

**Don't contradict global rules.** The agent sees both and may follow either. If a project genuinely needs different behavior, scope the global rule with a qualifier or use `CLAUDE.local.md`.

**Don't put secrets in committed files.** `.claude/settings.json` is committed. API keys, tokens, and passwords go in environment variables, `settings.local.json` (gitignored), or a secrets manager.

**Don't use rules/ for unconditional instructions.** A rule file without `paths:` frontmatter loads on every file read -- same as CLAUDE.md but harder to discover. Put unconditional rules in CLAUDE.md.

**Don't over-scope global permissions.** Global `permissions.allow` loads in every project. A Python-only developer doesn't need `javac` in their global allow list. Prefer project-level allows for language-specific tools.
