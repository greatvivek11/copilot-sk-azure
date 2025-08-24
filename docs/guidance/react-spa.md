# React SPA (Vite) Best Practices Guide

## Philosophy

* Treat React SPA as **UI layer only**; delegate business logic to APIs.
* Optimize for **developer experience** (fast builds, HMR, modular code).
* Prefer **composition over inheritance**.
* Embrace **TypeScript** for safety.
* Keep bundle size lean: code split, lazy load, tree-shake.

## Project Setup

* Use **Vite + React + TS** template.
* Configure absolute imports via `tsconfig.json` `paths`.
* Enforce linting & formatting: ESLint + Prettier.
* Add Husky + lint-staged for pre-commit checks.

```bash
npm create vite@latest my-app -- --template react-ts
```

## State Management

* Prefer **React Query (TanStack Query)** for server state.
* Use **Zustand / Jotai / Redux Toolkit** for global client state.
* Local UI state: keep inside components.

```tsx
const { data, isLoading } = useQuery(["todos"], fetchTodos);
```

## Routing

* Use **React Router v6+**.
* Lazy load routes via `React.lazy`.
* Prefer nested routing for structure.

```tsx
<Route path="/dashboard" element={<Dashboard />}>
  <Route path="stats" element={<Stats />} />
</Route>
```

## API Integration

* Abstract APIs in a **`/services`** layer.
* Use Axios or Fetch wrapper.
* Always type responses with DTOs.
* Handle errors centrally (React Query `onError`, Axios interceptors).

```ts
export const getUser = (id: string) => api.get<UserDto>(`/users/${id}`);
```

## Component Design

* Use **function components + hooks**.
* Break down UI into **atomic components** (atoms, molecules, organisms).
* Prefer **controlled components** for forms.
* Extract reusable UI to `/components/common`.

## Styling

* Options: TailwindCSS, CSS Modules, or Styled Components.
* Prefer **utility-first (Tailwind)** for speed + consistency.
* Keep design system in `/ui` with reusable components.

## Performance

* Use **React.memo** + `useCallback` / `useMemo` selectively.
* Avoid unnecessary context re-renders (split contexts).
* Code split heavy routes/components.
* Use **dynamic imports** for non-critical features.

```tsx
const Chart = React.lazy(() => import("./Chart"));
```

## Forms

* Use **React Hook Form** for scalable form state.
* Integrate with **Zod** for schema validation.
* Prefer controlled inputs with `Controller`.

```tsx
const schema = z.object({ email: z.string().email() });
const { control } = useForm({ resolver: zodResolver(schema) });
```

## Testing

* Unit: **Vitest** + React Testing Library.
* E2E: **Playwright** or Cypress.
* Test hooks in isolation, avoid DOM-heavy tests when possible.


## Deployment

* Output is static (`dist/`).
* Deploy to: **Azure Static Web Apps**, Vercel, Netlify, or GitHub Pages.
* Ensure proper SPA fallback (`index.html`) for routing.


## Security

* Never store tokens in localStorage → prefer **httpOnly cookies**.
* Use **CSP headers** & avoid `dangerouslySetInnerHTML`.
* Sanitize any dynamic HTML with DOMPurify if needed.


## Anti-Patterns

* Putting all state in Redux/global.
* Overusing `useEffect` (often signals misplaced logic).
* Fetching data inside deeply nested components without caching.
* Not splitting large components.
* Using index as React `key`.
* Inline functions in render without memoization (in perf-heavy lists).


## Tooling

* **Bundle Analyzer** (rollup-plugin-visualizer).
* **MSW (Mock Service Worker)** for API mocking in dev/test.
* **Storybook** for UI components.

## Extras

* Use **Error Boundaries** for resilience.
* Track perf with **React Profiler**.
* Consider **PWAs** if offline support needed.
