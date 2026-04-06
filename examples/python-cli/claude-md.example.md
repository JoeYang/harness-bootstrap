# My CLI Tool

A command-line data processing tool built with Click and httpx.

## Commands

| Action | Command |
|---|---|
| Build | `uv run build` |
| Test | `uv run pytest` |
| Test (single) | `uv run pytest tests/test_foo.py -k test_name` |
| Lint | `uv run ruff check src/ tests/` |
| Format | `uv run ruff format src/ tests/` |

## Architecture

- `src/my_cli/` -- cli.py (commands), client.py (httpx API client), models.py (Pydantic), config.py (env config)
- `tests/` -- pytest suite mirroring src/ structure

## Boundaries

### Always do
- Run `uv run pytest` before reporting work complete
- Run `uv run ruff format` before committing
- Create feature branches -- never commit to main
### Ask first
- Adding new dependencies to pyproject.toml
- Changing the CLI interface (command names, required args)
- Modifying CI/CD workflows
### Never do
- Push to main or master
- Modify .env files or committed secrets
- Disable or skip existing tests

## Security

See @.claude/rules/security.md
