# Dashboard App

A React + TypeScript analytics dashboard with chart visualizations and real-time data.

## Commands

| Action | Command |
|---|---|
| Build | `npm run build` |
| Dev | `npm run dev` |
| Test | `npm test` |
| Test (single) | `npm test -- --run src/components/__tests__/Chart.test.tsx` |
| Lint | `npx eslint src/` |
| Format | `npx prettier --write src/` |

## Architecture

- `src/components/` -- reusable UI components (charts, tables, filters)
- `src/pages/` -- route-level page components
- `src/hooks/` -- custom React hooks (useAuth, useQuery, useWebSocket)
- `src/api/` -- typed API client (fetch wrapper, request/response types)
- `src/__tests__/` -- vitest + React Testing Library suites mirroring src/

## Boundaries

### Always do
- Run `npm test` before reporting work complete
- Run `npx prettier --write` before committing
- Create feature branches -- never commit to main
### Ask first
- Adding new dependencies to package.json
- Changing route structure or navigation
- Modifying environment variable names
### Never do
- Push to main or master
- Modify .env files or committed secrets
- Disable or skip existing tests

## Security

See @.claude/rules/security.md
