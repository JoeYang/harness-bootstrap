---
paths: ["src/models/**", "src/db/**", "src/schema/**", "migrations/**", "alembic/**", "prisma/**"]
---
# Database Rules

## Migrations

- Migrations are always their own commit — never bundled with feature code
- Schema changes require review before implementation
- Never modify a migration file after it has been applied to any environment
- Every migration must be reversible — include a rollback/down step
- Name migrations descriptively: `add_index_users_email`, not `migration_042`

## Query Discipline

- Avoid N+1 queries — use eager loading, batch queries, or dataloaders
- Use transactions for multi-step data operations that must be atomic
- Never run unbounded queries — always include `LIMIT` or pagination
- Test with realistic data volumes, not empty tables — performance bugs hide in small datasets

## Schema Design

- Add indexes for columns used in `WHERE`, `JOIN`, and `ORDER BY` clauses
- Use foreign keys and constraints — enforce data integrity at the database level
- Prefer `NOT NULL` with explicit defaults over nullable columns
- Timestamp columns: always store in UTC, convert at the presentation layer

## Connection Management

- Never leak connections — use context managers, `try/finally`, or connection pool libraries
- Configure pool size based on expected concurrency; don't use unbounded pools
- Set statement timeouts to prevent runaway queries from holding connections
- Close connections explicitly in tests — don't rely on garbage collection
