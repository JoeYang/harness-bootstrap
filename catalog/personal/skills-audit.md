# Skills Audit

Source: `~/.claude/skills/*.md` (2 custom skills)

---

## /commit

**Where**: `~/.claude/skills/commit.md`
**What**: Stages files and creates a conventional commit with a well-formatted message.
**Invocation**: User-invocable slash command (`/commit`)

**Workflow**: `git status` + `git diff` -> `git log` (match style) -> stage specific files -> write `type(scope): description` message with body explaining WHY -> append co-author line. Does NOT push.

**Why this exists**: Without it, Claude either uses vague commit messages or stages everything with `git add -A`. This skill enforces the Conventional Commits format and specific-file staging from CLAUDE.md.
**Scope**: Global. Commit conventions are universal across all of Joe's projects.
**Overlap**: The `commit-commands` plugin in the marketplace covers similar ground. This custom skill may predate it; it matches Joe's exact co-author format (hardcodes "Claude Opus 4.6" regardless of active model — known quirk).

---

## /pr

**Where**: `~/.claude/skills/pr.md`
**What**: Creates a pull request from the current branch with a structured description.
**Invocation**: User-invocable slash command (`/pr`)

**Workflow**: `git log main..HEAD` + `git diff main...HEAD --stat` -> derive title (under 70 chars) -> `gh pr create` with Summary/Changes/Test plan sections.

**Why this exists**: Without it, Claude writes verbose PR descriptions or misses the test plan section. This skill enforces a consistent PR template.
**Scope**: Global. PR format is universal across all of Joe's projects.
**Overlap**: The `code-review` plugin provides a `/code-review` skill for reviewing PRs, but not for creating them. No direct conflict.
