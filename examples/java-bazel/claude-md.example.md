# Order Service

A Java order management service with REST API, persistence, and event publishing.

## Commands

| Action | Command |
|---|---|
| Build | `bazel build //...` |
| Test | `bazel test //...` |
| Test (single) | `bazel test //src/test/java/com/order:OrderServiceTest` |
| Format | `java -jar tools/google-java-format.jar --replace $(find src/ -name '*.java')` |
| Lint | `bazel build //... --aspects=@rules_lint//java:checkstyle.bzl%checkstyle` |

## Architecture

- `src/main/java/com/order/api/` -- REST controllers and request/response DTOs
- `src/main/java/com/order/service/` -- business logic and domain services
- `src/main/java/com/order/repository/` -- data access layer (JPA repositories)
- `src/main/java/com/order/event/` -- event publishing (Kafka producers)
- `src/test/java/` -- JUnit 5 suites mirroring src/main/java/ structure

## Boundaries

### Always do
- Run `bazel test //...` before reporting work complete
- Run google-java-format before committing
- Create feature branches -- never commit to main
### Ask first
- Adding new dependencies to BUILD.bazel or WORKSPACE
- Changing database schema or migrations
- Modifying Kafka topic names or event schemas
### Never do
- Push to main or master
- Modify .env files or committed secrets
- Disable or skip existing tests

## Security

See @.claude/rules/security.md
