---
paths: ["src/**/__tests__/**/*.ts", "src/**/__tests__/**/*.tsx", "src/**/*.test.ts", "src/**/*.test.tsx"]
---

## Testing

TDD workflow: write failing test -> implement -> pass -> refactor -> re-run.

Use vitest + React Testing Library for all tests. Coverage target: 80% line coverage. Run `npm test -- --coverage`.

Failure injection -- every feature must include tests for:
- Network: fetch timeouts, HTTP 4xx/5xx responses, malformed JSON
- Inputs: empty props, missing required fields, invalid form data
- Rendering: loading states, error boundaries, empty data sets
- Auth: expired tokens, unauthorized access, session timeouts

Use `vi.mock()` for module mocks. Use `msw` for HTTP request mocking -- never hit real APIs in tests. Use `screen.getByRole` over `getByTestId` for accessibility.

Bug fixes require a failing regression test first. Never disable or skip tests -- fix them.
