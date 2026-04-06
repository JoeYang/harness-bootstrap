# Exchange Gateway

A low-latency exchange connectivity gateway for APAC markets. FIX/ITCH protocol translation, order routing, and market data distribution.

slack_channel: #trading-dev

## Commands

| Action | Command |
|---|---|
| Build | `bazel build //...` |
| Test | `bazel test //...` |
| Test (single) | `bazel test //src/gateway:gateway_test` |
| Format | `clang-format -i $(find src/ -name '*.cpp' -o -name '*.h')` |
| Lint | `clang-tidy src/**/*.cpp -- -std=c++20` |
| Benchmark | `bazel run //bench:latency_bench -- --benchmark_format=json` |

## Architecture

- `src/gateway/` -- FIX session management, order routing, connection lifecycle
- `src/feed/` -- market data ingestion (ITCH/MDP), book building, distribution
- `src/proto/` -- shared protobuf definitions for internal messaging
- `test/` -- gtest suites mirroring src/; `bench/` -- latency benchmarks
- `sim/` -- exchange simulator for integration testing

## Boundaries

### Always do
- Run `bazel test //...` before reporting work complete
- Run `clang-format` before committing
- Run `./smoke_test_all.sh <exchange>` before marking an exchange done
- Create feature branches -- never commit to main
### Ask first
- Adding new third-party dependencies to WORKSPACE
- Changing protobuf message definitions
- Modifying the hot path (gateway/feed critical sections)
### Never do
- Push to main or master
- Modify .env files or committed secrets
- Disable or skip existing tests
- Allocate on the heap in hot path code

## Rules

See @.claude/rules/security.md, @.claude/rules/cpp.md, @.claude/rules/testing.md
