# Rules Cookbook

Copy-paste recipes for `.claude/rules/*.md`. Each recipe is a complete file.

## Path-Scoped Frontmatter

Rules with YAML frontmatter load conditionally; without frontmatter they load every turn.
```yaml
---
paths: ["src/api/**/*.ts"]
---
```
`**` matches directories, `*` matches files. Multiple patterns are OR'd.

## `.claude/rules/testing.md`

```markdown
## Testing
TDD workflow: write failing test -> implement -> pass -> refactor -> re-run.

Failure injection -- every feature must include tests for:
- Network: timeouts, connection refused, partial responses
- Dependencies: database down, service unavailable
- Inputs: empty, null, oversized, wrong type
- Resources: memory, disk, thread pool exhaustion
- Concurrency: race conditions on shared state

Coverage: every branch, edge case, and boundary. Never disable or skip tests -- fix them. Bug fixes require a failing regression test first.
```

## `.claude/rules/security.md`

Two tiers. Pick standard (OWASP top 10) or strict (+ crypto/audit/supply chain).
```markdown
## Security
- No hardcoded secrets -- read from env vars or secrets manager
- Validate all input at system boundaries
- No SQL injection, XSS, CSRF, command injection, path traversal
- Least privilege for roles, tokens, and database access
- Auth checks on every protected route (authn + authz)
- Never log passwords, tokens, or PII
<!-- strict tier: add these three lines -->
- Crypto: standard libs only (AES-256, RSA-2048+, bcrypt/argon2), no custom crypto
- Audit log all auth events, privilege changes, and data access
- No deps with known CVEs; flag unmaintained packages for review
```

## `.claude/rules/boundaries.md`

The always/ask/never pattern (from GitHub's analysis of 2,500 repos).
```markdown
## Boundaries
### Always do
- Run tests before reporting work complete
- Run the formatter before committing
- Create feature branches -- never commit to main
### Ask first
- Adding new dependencies
- Changing database schemas or migrations
- Modifying CI/CD configuration
- Deleting or renaming public API surfaces
### Never do
- Push to main or master
- Modify .env files or committed secrets
- Disable or skip existing tests
- Force-push to shared branches
```

## `.claude/rules/cpp.md`

```markdown
---
paths: ["**/*.cpp", "**/*.h", "**/*.cc", "**/*.hpp"]
---
- RAII for all resource management -- no manual new/delete
- `unique_ptr`/`shared_ptr` when heap allocation is needed
- `const` by default: params, member functions, locals
- C++17 minimum; C++20 (concepts, ranges) when available
- `#pragma once` over include guards
- `string_view` over `const string&` for read-only params
```

## `.claude/rules/python.md`

```markdown
---
paths: ["**/*.py"]
---
- Type hints on all signatures and return types
- `dataclass`/`BaseModel` over raw dicts for structured data
- `__all__` in every `__init__.py`
- `ruff` for formatting and linting
- `pathlib.Path` over `os.path`
- `from __future__ import annotations` for forward refs
```

## `.claude/rules/typescript.md`

```markdown
---
paths: ["**/*.ts", "**/*.tsx"]
---
- Strict mode (`"strict": true` in tsconfig)
- Never use `any` -- `unknown` + type guards
- `interface` over `type` for object shapes
- Named exports only -- no default exports
- `readonly` on immutable properties
- `Map`/`Set` over plain objects for dynamic keys
```

## `.claude/rules/java.md`

```markdown
---
paths: ["**/*.java"]
---
- `record` for immutable data (Java 16+)
- `sealed` classes for closed hierarchies (Java 17+)
- Unchecked exceptions over checked in new code
- google-java-format for formatting
- `Optional` returns over null; never pass Optional as param
- `var` for locals when type is obvious from RHS
```
