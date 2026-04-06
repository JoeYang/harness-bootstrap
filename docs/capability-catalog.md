# Capability Catalog v1

Maps project signals to harness capabilities. The `/harness-bootstrap` skill reads this at runtime to adapt its questions per project. The `/harness-update` skill diffs installed capabilities against this catalog to offer incremental upgrades.

---

## Three Tiers

1. **Auto-included** -- signal detected, no question needed (permissions, deny list, stop check)
2. **Confirm/adjust** -- signal detected, user confirms or tweaks (build commands, coverage target)
3. **Custom on-the-fly** -- no catalog match, LLM generates rules from project context

---

## CLAUDE.md Capabilities

| Capability | Signal (auto-detect) | Question (if needed) | Generates |
|---|---|---|---|
| Project identity | Always | "Project name + one-line description?" | CLAUDE.md header |
| Build commands | `BUILD.bazel`, `package.json`, `pyproject.toml`, `pom.xml`, `Cargo.toml` | "Detected {tool}. Correct?" | `# Commands` section |
| Test commands | `*_test.*`, `test_*.py`, `conftest.py`, `jest.config.*`, `vitest.config.*`, `.rspec` | "Detected {framework}. Correct?" | `# Commands` section |
| Lint/format commands | `.prettierrc`, `ruff` in `pyproject.toml`, `.clang-format`, `checkstyle.xml`, `.eslintrc.*` | "Detected {tool}. Correct?" | `# Commands` section |
| Architecture notes | `src/`, `lib/`, `pkg/`, `cmd/` directory structure | "Brief architecture?" (optional) | `# Architecture` section |
| Boundaries (always do) | Build + test + format commands detected | Auto-generated from detected commands | `# Boundaries` section |
| Boundaries (ask first) | Always | Pre-filled defaults, user adjusts | `# Boundaries` section |
| Boundaries (never do) | Always | Pre-filled defaults, user adjusts | `# Boundaries` section |
| Slack channel | Global config has slack rules | "Notification channel?" | `slack_channel` in CLAUDE.md |

---

## Settings.json Capabilities

| Capability | Signal (auto-detect) | Question (if needed) | Generates |
|---|---|---|---|
| Language permissions | Detected language + build tool | Auto (no question) | `permissions.allow` |
| Credential deny list | Always | Auto-included | `permissions.deny` |
| Auto-format hook | `.prettierrc`, `.clang-format`, `ruff` in `pyproject.toml` | "Enable auto-format on save?" | `hooks.PostToolUse` |
| Auto-lint hook | `.eslintrc.*`, `ruff` in `pyproject.toml`, `checkstyle.xml` | "Enable auto-lint on save?" | `hooks.PostToolUse` |
| Branch protection hook | `.git/` directory detected | "Block direct edits on main/master?" | `hooks.PreToolUse` |
| Stop check hook | Always | Auto-included | `hooks.Stop` |

---

## Rules Capabilities

| Capability | Signal (auto-detect) | Question (if needed) | Generates |
|---|---|---|---|
| Testing rules | `conftest.py`, `jest.config.*`, `*_test.*`, `BUILD.bazel` with test targets | "Coverage target? Failure injection level?" | `rules/testing.md` |
| Security rules | Always | "Standard (OWASP) / strict (+ crypto, audit) / minimal?" | `rules/security.md` |
| Language: Python | `*.py`, `pyproject.toml` | "Include Python rules?" | `rules/python.md` |
| Language: TypeScript | `*.ts`, `*.tsx`, `tsconfig.json` | "Include TypeScript rules?" | `rules/typescript.md` |
| Language: C++ | `*.cpp`, `*.h`, `BUILD.bazel` with `cc_*` rules | "Include C++ rules?" | `rules/cpp.md` |
| Language: Java | `*.java`, `pom.xml`, `BUILD.bazel` with `java_*` rules | "Include Java rules?" | `rules/java.md` |
| Agent team orchestration | NOT auto-detectable | "Does this project use agent teams?" | `rules/agent-teams.md` |
| Exchange conventions | `smoke_test_all.sh`, or user indicates trading project | "Trading/exchange project?" | `rules/exchange.md` |
| Trading latency rules | Exchange capability = yes | Auto-included with exchange | `rules/trading-latency.md` |
| API design rules | `fastapi` in deps, `express` in `package.json`, `.proto` + `grpc` in deps | "Include API design rules?" | `rules/api-design.md` |
| Database rules | `alembic/`, `migrations/`, ORM in deps (`sqlalchemy`, `prisma`, `typeorm`) | "Include database rules?" | `rules/database.md` |
| Frontend rules | `react` in `package.json`, `vue` in deps, `svelte` in deps, `next.config.*` | "Include frontend rules?" | `rules/frontend.md` |

---

## Domain-Specific Capabilities

These extend the base catalog for specialized project types.

| Capability | Signal (auto-detect) | Question (if needed) | Generates |
|---|---|---|---|
| gRPC / protobuf rules | `*.proto` files, `grpc` in deps | "Include protobuf rules?" | `rules/grpc.md` |
| WebSocket rules | `ws` or `socket.io` in deps, `websocket` in imports | "Include connection lifecycle rules?" | `rules/websocket.md` |
| CI/CD awareness | `.github/workflows/`, `Jenkinsfile`, `.gitlab-ci.yml` | "Include CI rules in boundaries?" | CLAUDE.md boundaries |
| Docker rules | `Dockerfile`, `docker-compose.yml`, `compose.yaml` | "Include container rules?" | `rules/docker.md` |
| Terraform / IaC rules | `*.tf` files, `cdk.json`, CDK stack files | "Include infrastructure-as-code rules?" | `rules/iac.md` |

---

## Tier Assignment Summary

| Tier | Capabilities |
|---|---|
| 1 -- Auto-included | Language permissions, credential deny list, stop check hook, boundaries (always/never defaults), trading latency (when exchange = yes) |
| 2 -- Confirm/adjust | Build/test/format commands, architecture notes, auto-format hook, auto-lint hook, branch protection, testing rules, security level, language rules, all domain-specific |
| 3 -- Custom on-the-fly | Any signal not in this catalog -- LLM reads project context and generates custom rules at bootstrap time |

---

## Extensibility

Adding a new capability:

1. Add a row to the appropriate table above (CLAUDE.md, Settings, Rules, or Domain-Specific)
2. Specify the signal (file patterns, dependency names, or "Always" / "NOT auto-detectable")
3. Write a question template for Tier 2, or mark as auto-included for Tier 1
4. If the capability needs a reusable rule file, add it to `rules-library/`
5. Bump the version number at the top of this file

The `/harness-bootstrap` and `/harness-update` skills pick up new rows on their next run -- no code changes needed.
