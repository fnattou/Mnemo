# Frontend Architecture & React Design

## Decision Summary
TaskBoard frontend uses **React 18 with TypeScript and Zustand for state management**, leveraging strict typing to catch bugs at build time and Zustand's lightweight model preventing Redux boilerplate.

## React + TypeScript Rationale
React's component model aligns with TaskBoard's UI: independent task cards, reusable project panels, and dynamic filters. TypeScript provides compiler-verified prop types, preventing silent failures from API contract changes. Next.js adds server-side rendering for initial page load optimization and SEO compatibility for shared task links.

## State Management with Zustand
Zustand replaces Redux, providing:
- **Minimal boilerplate**: Stores defined as functions returning state and setters, not action creators
- **No provider nesting hell**: Direct hook imports without Redux context layers
- **Derived state**: Computed selectors prevent unnecessary re-renders
- **DevTools**: Redux DevTools compatible; inspect state snapshots across sessions

Store structure mirrors domain: TaskStore (CRUD operations), FilterStore (current view), UserStore (authentication state). Async operations handled via middleware, not wrapped in thunks.

## Component Architecture
- **Page components**: Route-level, fetch data, manage layout
- **Container components**: Business logic, coordinate multiple feature components
- **Presentational components**: Pure UI, accept props, no side effects
- **Hooks**: Custom hooks extract repetitive logic (useTask, useFilteredTasks)
- **Strict TypeScript**: No `any` types; discriminated unions for variant components

## Build Tooling
- Vite for development (50ms HMR vs Webpack's 1-2s), esbuild for production bundling
- ESLint with react-hooks plugin enforces dependency rules
- Vitest for unit tests; React Testing Library for component tests

## Alternatives Considered
1. **Vue**: Smaller ecosystem; harder hiring in competitive market
2. **Redux**: Industry standard but verbose; Zustand's simplicity fits our needs
3. **Vanilla TypeScript with Web Components**: Reinventing component model; framework value > cost

## Performance Optimization
- Code splitting by route using React.lazy, reducing main bundle 60%
- Memoization (React.memo, useMemo) only on demonstrable bottlenecks, avoiding premature micro-optimization
- Image optimization: WebP with JPEG fallback, responsive srcset for different devices

## Critical Notes
- Controlled components for form inputs enable real-time validation feedback
- Error boundaries catch component crashes, preventing white screens
- Accessibility (a11y) via semantic HTML and ARIA labels; keyboard navigation for all workflows
