# Global CLAUDE.md Audit

Reference for trimming `~/.claude/CLAUDE.md` from 224 lines to ~90-100 lines (55% reduction).

Source: 224 lines across 16 sections. Classified into: 11 keep global, 2 move entirely, 3 split.

## Keep Global (~90 lines)

These are universal rules that apply to every project regardless of language, domain, or team size.

| Section | Lines | Reason |
|---|---|---|
| Build System | 3-17 | Language-to-tool mapping -- universal |
| Project Planning | 18-24 | Researcher review, ASCII diagrams -- all non-trivial projects |
| Testing | 25-39 | TDD, coverage, failure injection -- universal philosophy |
| Documentation | 41-55 | README/docs sync rules -- universal |
| Scope | 56-57 | Focus on cwd -- universal |
| Security | 59-68 | OWASP, crypto, secrets -- universal |
| What the Agent Gets Wrong | 70-76 | Guardrails against common LLM mistakes -- universal |
| Commit Rules + format | 98-121 | Commit discipline, conventional commits -- universal |
| Model Selection | 123-141 | Opus/Sonnet/Haiku routing -- universal (trim "human review" to project) |
| Error Recovery | 142-147 | 3-attempt limit, escalation -- universal |
| Context Management | 149-153 | `/compact` discipline -- universal |
| Session Memory | 217-224 | MEMORY.md read/write -- universal |

## Move to Project Level (~95 lines)

These are domain-specific or team-configuration rules that should only load in projects that need them.

| Section | Lines | Destination |
|---|---|---|
| Agent Team Orchestration | 155-215 (60 lines) | `rules-library/agent-teams.md` -> project `.claude/rules/` |
| Exchange Smoke Test Gate | 94-96 (3 lines) | `rules-library/exchange.md` -> exchange projects only |

## Split Between Global and Project (~33 lines)

These sections have universal principles that stay global, plus project-specific details that move.

| Section | Global (stays) | Project (moves) |
|---|---|---|
| Branching Strategy | "Never commit to main" (line 83-85) | Worktree workflow, review tiers (lines 86-92) |
| Model Selection | Opus/Sonnet/Haiku table (lines 123-136) | "Human review mandatory" list (lines 137-141) |
| Slack Notifications | Default to #general (line 80) | Project-specific channel override (lines 78-81) |

## Summary

| Category | Lines | Sections |
|---|---|---|
| Keep global | ~90 | 11 |
| Move to project (entirely) | ~63 | 2 |
| Split (global + project) | ~33 | 3 |
| **Total** | **224** | |
| **After trim** | **~90-100** | **55% reduction** |
