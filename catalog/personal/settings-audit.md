# Settings.json Audit

Source: `~/.claude/settings.json` + `~/.claude/settings.local.json`

---

## Permissions — Allow List

**What**: Pre-approved tool invocations that skip the permission prompt.
**Where**: `permissions.allow[]`
**Value**: 20 Bash commands (git, bazel, ls, npm, npx, node, tsc, ts-node, tsx, python, python3, pip, pip3, uv, java, javac, mvn, gradle, curl, wget) + WebSearch + 29 WebFetch domains.

**Why this exists**: Without it, every `git status` or `npm install` triggers a permission prompt. Blocks flow in sessions with hundreds of tool calls.
**When useful**: Every project. The language-specific commands (java, gradle, mvn) are harmless when unused.
**When NOT**: Never harmful — unused permissions are inert. But project-level settings could be tighter (Python project doesn't need javac).
**Candidate for**: Both. Global keeps the full superset; project-level could narrow to relevant tools only.

**WebFetch domains**: Dev docs (Python, npm, MDN, SO), cloud (AWS, GCP, Docker), Anthropic, and finance (FIX, Aeron, SEC, OpenFIGI, QuickFIX/J, Yahoo Finance, Bloomberg, Reuters). Finance domains only matter for trading projects but are harmless globally.

---

## Permissions — Deny List

**Where**: `permissions.deny[]`
**Value**: `Bash(rm -rf *)`, `Read(.env*)`
**Why**: Prevents catastrophic deletions and secret leakage (.env files contain API keys).
**Candidate for**: Global only. Universal safety rails.

---

## Environment Variables

**Where**: `env{}`

### CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS = `"1"`
**Why**: Enables multi-agent team orchestration. Required for `teammateMode` to function.
**When useful**: Complex projects with parallelizable work. Harmless when unused.
**Candidate for**: Global only (feature flag).

### GITHUB_TOKEN = `ghp_...` (hardcoded — **SECURITY: rotate and externalize**)
**Why**: Enables `gh` CLI and GitHub MCP without repeated auth.
**Risk**: Live personal access token hardcoded in a config file. If this file is ever committed, shared, or read by a compromised tool, full GitHub access is exposed. **Action required**: rotate this token immediately, then use `gh auth token` or a keyring/secrets manager.
**Candidate for**: Global only, but the value MUST be externalized.

---

## MCP Servers

**What**: Model Context Protocol servers providing external tool integrations.
**Where**: `mcpServers{}`

### sentry — `http://localhost:8585/sse`
**Why**: Error tracking context during debugging.
**When useful**: Projects with Sentry. **When NOT**: Fails silently when not running, but may add latency to session startup during connection timeout. Most projects don't use Sentry.
**Candidate for**: Project-level only.

### slack — `https://mcp.slack.com/mcp` (Bearer token **SECURITY: rotate and externalize**)
**Why**: Slack integration (post messages, read channels).
**Risk**: `xoxb-` prefixed token = Slack bot token with full API access. Hardcoded and live. **Action required**: rotate this token, then store in environment variable or secrets manager.
**When useful**: Team projects. **When NOT**: Solo/learning projects.
**Candidate for**: Global (cross-project), but the token MUST be externalized.

### sim-exchange — stdio server connecting to `localhost:50051`
**Why**: MCP tools for the trading simulator (order entry, market data).
**When useful**: Only exchange-connectivity and agency-trading development.
**When NOT**: Every other project. Spawns a Python process on session start even when unused. Broken venv/path fails silently and wastes startup time. Command path is hardcoded to `/home/joeyang/Claude-code/sim-exchange/mcp/.venv/bin/python` — breaks on any other machine.
**Candidate for**: **Project-level only** (in `.mcp.json`). Strongest extraction candidate — heavyweight stdio server that only 2-3 projects need.

---

## Feature Flags

| Flag | Value | Why | Candidate |
|---|---|---|---|
| `effortLevel` | `"high"` | Longer, more thorough reasoning | Global (personal preference) |
| `skipDangerousModePermissionPrompt` | `true` | Skip dangerous-mode prompt; relies on deny list | Global (risk-tolerant preference) |
| `teammateMode` | `"auto"` | Auto-spawn teammates; needs `AGENT_TEAMS=1` | Global (harmless when unused) |

---

## Enabled Plugins

**What**: Plugin suites providing additional skills and capabilities.
**Where**: `enabledPlugins{}`

| Plugin | Suite | Purpose |
|---|---|---|
| code-review | claude-plugins-official | PR review skill |
| frontend-design | claude-plugins-official | UI/UX generation |
| code-simplifier | claude-plugins-official | Refactoring suggestions |
| github | claude-plugins-official | GitHub integration tools |
| playwright | claude-plugins-official | Browser automation/testing |
| superpowers | claude-plugins-official | TDD, plans, brainstorming, worktrees, debugging |
| claude-md-management | claude-plugins-official | CLAUDE.md audit/improvement |
| slack | claude-plugins-official | Slack messaging skills |
| skill-creator | claude-plugins-official | Create/edit custom skills |
| ralph-loop | claude-plugins-official | Recurring task loops |
| financial-analysis | financial-services-plugins | DCF, LBO, comps, 3-statement models |
| equity-research | financial-services-plugins | Earnings analysis, sector reports |
| private-equity | financial-services-plugins | Deal screening, IC memos |
| wealth-management | financial-services-plugins | Portfolio rebalancing, client reports |

**Candidate for**: Both. Core plugins (superpowers, code-review, github, claude-md-management) are universal. Financial plugins (4 suites) add 40+ skills that are noise in non-finance sessions — candidate for project-level.

---

## Local Overrides (`settings.local.json`)

**What**: Machine-specific permission overrides that stack on top of `settings.json`.
**Where**: `~/.claude/settings.local.json` → `permissions.allow[]`
**Value**: 8 additional allow entries.

| Permission | Purpose | Status |
|---|---|---|
| `mcp__...__slack_send_message` | Slack MCP send permission | Active — needed for Slack integration |
| `Bash(nvidia-smi)` | GPU diagnostics | Stale — from a one-off debugging session |
| `Bash(lspci)` | PCI device listing | Stale — GPU driver investigation |
| `Bash(ubuntu-drivers devices:*)` | Driver detection | Stale — GPU driver investigation |
| `Bash(dpkg -l)` | Package listing | Stale — dependency investigation |
| `Read(//proc/driver/nvidia/**)` | NVIDIA driver info | Stale — GPU diagnostics |
| `Read(//tmp/**)` | Read temp files | **Risk**: broad access to /tmp which may contain secrets from other processes |
| `Bash(identify /tmp/nyan-cat-icon.png)` | ImageMagick on one file | Stale — one-off image inspection |

**Action required**: Clean up stale entries. Only `slack_send_message` is actively needed. `Read(//tmp/**)` is a security concern — overly broad.
**Candidate for**: Global only (machine-specific). Clean up before sharing or cataloging further.
