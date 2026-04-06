# Exchange Implementation Rules

## Smoke Test Gate

Every exchange implementation MUST pass `./smoke_test_all.sh <exchange>` before being considered done. Do not skip this step or mark the task complete without a passing run.

The smoke test validates the full pipeline end-to-end:
1. Simulator startup — exchange sim process launches and accepts connections
2. Order entry — orders are submitted and acknowledged
3. Matching — orders match and generate fills
4. Market data — quotes and trades publish to the feed
5. Observer display — the observer UI renders live data correctly

If any stage fails, consult `.claude/skills/exchange-smoke-test.md` for the debug guide and common failure modes.

## Session Data Conventions

- APAC exchange sessions follow their local trading calendars and timezones
- Use exchange-native instrument symbology (not normalized/internal IDs)
- Respect session boundaries: pre-open, continuous trading, closing auction
- Log all gateway messages at DEBUG level during development for replay capability
