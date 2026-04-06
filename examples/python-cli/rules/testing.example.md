---
paths: ["tests/**/*.py"]
---

## Testing

TDD workflow: write failing test -> implement -> pass -> refactor -> re-run.

Coverage target: 80% line coverage. Run `uv run pytest --cov=src/my_cli --cov-fail-under=80`.

Failure injection -- every feature must include tests for:
- Network: httpx timeouts, connection refused, HTTP 429/500 responses
- Inputs: empty strings, missing required fields, oversized payloads
- Config: missing env vars, malformed config values
- File I/O: missing files, permission denied, disk full

Use `pytest.raises` for expected exceptions. Use `respx` or `pytest-httpx` to mock HTTP calls -- never hit real APIs in tests.

Bug fixes require a failing regression test first. Never disable or skip tests -- fix them.
