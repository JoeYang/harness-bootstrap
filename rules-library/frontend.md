---
paths: ["src/components/**", "src/pages/**", "src/views/**", "src/app/**", "src/routes/**/*.tsx", "src/routes/**/*.vue", "src/routes/**/*.svelte"]
---
# Frontend Rules

## Component Patterns

- Functional components only (React) — no class components
- State management: prefer hooks (React), composables (Vue), or stores (Svelte) for local state; global stores only for truly shared state
- Memoize expensive computations: `useMemo`/`useCallback` (React), `computed` (Vue)
- Component naming: `PascalCase` for components, `camelCase` for hooks and utilities
- Keep components focused — extract logic into custom hooks/composables when a component exceeds ~150 lines

## Async & Loading States

- Every async operation must handle three states: loading, error, and empty/success
- Show meaningful loading indicators — never leave the user staring at a blank screen
- Error states must include a recovery action (retry button, navigation, or clear message)

## Accessibility

- Semantic HTML first: `<button>`, `<nav>`, `<main>`, `<article>` over generic `<div>`
- ARIA labels on interactive elements that lack visible text
- Keyboard navigation: all interactive elements must be reachable and operable via keyboard
- Color contrast: meet WCAG AA minimum (4.5:1 for normal text)

## Styling

- Prefer CSS modules or Tailwind over global stylesheets — avoid class name collisions
- Responsive design: mobile-first approach with progressive enhancement for larger screens
- No inline styles except for truly dynamic values (e.g., computed positions)

## Testing

- Test component rendering and user interaction, not implementation details
- Use `screen.getByRole` / `getByLabelText` over `getByTestId` — test what the user sees
- Mock network requests, not internal functions
