# Agents Audit

Source: `~/.claude/agents/*.md` (8 agents)

---

## architecture-explainer (Sonnet)

**What**: Reads codebases and produces structured architectural summaries.
**Where**: `~/.claude/agents/architecture-explainer.md`
**Model**: Sonnet -- **Why**: Read-heavy analysis; Sonnet is cost-effective for structured output.
**Tools**: Read, Glob, Grep, Bash, WebSearch
**Why this exists**: Without a specialist, Claude skims rather than systematically mapping entry points, data flows, and abstractions.
**When useful**: New repo onboarding, legacy code exploration. **When NOT**: Projects you already know.
**Candidate for**: Global agent.

---

## doc-sync (Sonnet)

**What**: Updates README.md and CLAUDE.md to match code changes after commits.
**Where**: `~/.claude/agents/doc-sync.md`
**Model**: Sonnet -- **Why**: Surgical text edits from diffs. No deep reasoning required.
**Tools**: Default (no restrictions — only agent without explicit allowedTools)
**Why this exists**: Documentation drift is the default. Without post-commit sync, README diverges from code within weeks.
**When useful**: Every project with docs. **When NOT**: Projects with no README/CLAUDE.md yet (syncs, not authors).
**Candidate for**: Global agent. Overlaps with claude-md-management plugin but broader (covers README).

---

## low-latency-trading-engineer (Opus)

**What**: Designs/implements performance-critical trading infra (C++/Java, lock-free, kernel bypass).
**Where**: `~/.claude/agents/low-latency-trading-engineer.md`
**Model**: Opus -- **Why**: Deep reasoning about cache lines, memory ordering, nanosecond trade-offs. Wrong answers are expensive.
**Tools**: Default (all)
**Why this exists**: Low-latency code has non-obvious correctness requirements (false sharing, GC pressure) that generalists miss.
**When useful**: Trading system implementation, latency optimization. **When NOT**: Non-trading C++. Opus cost unjustified.
**Candidate for**: Project-level only (trading repos).

---

## react-web-developer (Sonnet)

**What**: Builds React frontends with gRPC/REST integration and git workflows.
**Where**: `~/.claude/agents/react-web-developer.md`
**Model**: Sonnet -- **Why**: Standard implementation work. React patterns are well-established.
**Tools**: Default (all)
**Why this exists**: Enforces consistent patterns (TypeScript strict, React Query, error/loading states, accessibility).
**When useful**: React projects with API integrations. **When NOT**: Non-React or backend-only. Only 2-3 of 17 projects are React.
**Candidate for**: Project-level only (React repos).

---

## read-only-researcher (Sonnet)

**What**: Investigates code without making any modifications.
**Where**: `~/.claude/agents/read-only-researcher.md`
**Model**: Sonnet -- **Why**: Read-heavy exploration; Sonnet sufficient for search and synthesis.
**Tools**: Glob, Grep, Read, WebFetch, WebSearch (NO write tools -- that is the point)
**Why this exists**: Safe to launch in production codebases. The restricted tool set prevents accidental modifications.
**When useful**: Pre-refactoring analysis, tracing bugs. **When NOT**: When you need to actually fix something.
**Candidate for**: Global agent (universal safety pattern).

---

## strict-code-reviewer (Sonnet)

**What**: Line-by-line code review: typos, formatting, naming, code smells.
**Where**: `~/.claude/agents/strict-code-reviewer.md`
**Model**: Sonnet -- **Why**: Pattern matching and checklist application. No deep reasoning needed.
**Tools**: Default (all)
**Why this exists**: Self-review misses blind spots. A separate reviewer with an explicit checklist catches what the author missed.
**When useful**: Post-implementation review, pre-commit gate. **When NOT**: Trivial single-line fixes.
**Candidate for**: Global agent. Referenced in CLAUDE.md branching strategy for agent-led reviews.

---

## sys-troubleshooter (Sonnet)

**What**: Diagnoses system/application failures with structured hypothesis-driven methodology.
**Where**: `~/.claude/agents/sys-troubleshooter.md`
**Model**: Sonnet -- **Why**: Diagnostic methodology is systematic, not creative. Sonnet follows the framework well.
**Tools**: Default (all)
**Why this exists**: Enforces 5-phase diagnostic process (symptoms, hypothesis, evidence, confirmation, remediation) instead of random changes.
**When useful**: Crashes, performance degradation, network issues, build failures. **When NOT**: Obvious error messages.
**Candidate for**: Global agent.

---

## trading-systems-lead (Opus)

**What**: Team lead coordination for trading system delivery: requirements, task breakdown, testing oversight.
**Where**: `~/.claude/agents/trading-systems-lead.md`
**Model**: Opus -- **Why**: Architectural trade-offs, latency budget allocation, cross-component coordination require deep reasoning.
**Tools**: Default (all)
**Why this exists**: Decomposes trading work with latency budgets, exchange protocol awareness, and measurable acceptance criteria.
**When useful**: Planning trading features, reviewing architecture. **When NOT**: Non-trading projects entirely.
**Candidate for**: Project-level only (trading repos).
