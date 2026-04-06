## Testing

TDD workflow: write failing test -> implement -> pass -> refactor -> re-run.

Use gtest/gmock for all tests. Structure: one `*_test.cpp` per source file in `test/`.

Failure injection -- every component must include tests for:
- Network: socket timeouts, connection refused, partial reads, malformed packets
- Protocol: invalid FIX tags, out-of-sequence messages, session resets
- Resources: buffer overflow attempts, queue full, thread pool exhaustion
- Concurrency: race conditions on shared order state

Latency benchmarks: use Google Benchmark for hot path functions. Regressions in p99 latency fail the build. Run `bazel run //bench:latency_bench` to verify.

Never disable or skip tests -- fix them. Bug fixes require a failing regression test first.
