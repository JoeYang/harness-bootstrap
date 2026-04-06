# Trading Latency Rules

## Latency Budgets

Enforce these targets for all trading system components:
- Order entry path: < 50 microseconds (wire to ack)
- Market data processing: < 10 microseconds (packet to internal update)
- End-to-end tick-to-trade: < 500 microseconds

## Hot Path Rules

The critical path (market data ingestion, signal evaluation, order submission) must follow these constraints:
- No heap allocation — pre-allocate all buffers at startup
- No locks — use lock-free structures or single-threaded ownership
- No syscalls — avoid logging, file I/O, or network calls that block
- No exceptions — use error codes or expected-value types on the hot path

## C++ Specifics

- Prefer stack allocation and arena allocators over `new`/`malloc`
- Avoid virtual dispatch in hot path — use CRTP or compile-time polymorphism
- Mark hot-path functions with `__attribute__((hot))` and cold error paths with `__attribute__((cold))`
- Use `likely()`/`unlikely()` branch hints where profiling confirms skew

## Measurement Discipline

- Never assume a change improves performance — always measure before and after
- Benchmark every optimization with realistic market data replay
- Profile with `perf stat` and `perf record` before optimizing — fix the measured bottleneck, not the guessed one
- Track latency percentiles (p50, p99, p99.9), not averages — tail latency kills
