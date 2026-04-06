---
paths: ["src/test/**/*.java"]
---

## Testing

TDD workflow: write failing test -> implement -> pass -> refactor -> re-run.

Use JUnit 5 + Mockito for all tests. Coverage target: 80% line coverage. One test class per source class in `src/test/java/`.

Failure injection -- every service must include tests for:
- Network: HTTP client timeouts, connection refused, 5xx from downstream services
- Database: connection pool exhaustion, constraint violations, deadlocks
- Messaging: Kafka broker unavailable, serialization failures, duplicate messages
- Inputs: null fields, empty strings, boundary values, malformed JSON

Use `@ExtendWith(MockitoExtension.class)` for unit tests. Use `@SpringBootTest` sparingly -- only for integration tests. Use Testcontainers for database and Kafka integration tests.

Bug fixes require a failing regression test first. Never disable or skip tests -- fix them.
