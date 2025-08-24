# React SSR (Next.js) Best Practices Guide

## Philosophy

* Treat **Next.js as a full-stack framework**, not just SSR.
* Balance **static generation (SSG)**, **server-side rendering (SSR)**, and **client-side rendering (CSR)**.
* Prefer **edge rendering** when latency is critical.
* Optimize for **performance, SEO, and developer experience**.

## Project Setup

* Use **TypeScript** from the start.
* Enforce linting/formatting with **ESLint + Prettier**.
* Organize by **feature-based folder structure** (`/features/*`, `/components/*`).
* Use **absolute imports & path aliases** via `tsconfig.json`.

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@components/*": ["components/*"]
    }
  }
}
```

## Data Fetching Strategy

* Prefer **`getStaticProps` + ISR** for mostly static content.
* Use **`getServerSideProps`** only when personalization/real-time data is required.
* Fetch data in **React Query / SWR** for client hydration.
* Use **API Routes** or dedicated backend for complex logic.

```ts
export async function getStaticProps() {
  const posts = await getAllPosts();
  return { props: { posts }, revalidate: 60 };
}
```

## Routing & Navigation

* Use **App Router** (Next.js 13+) for layouts, nested routes, server components.
* Use **dynamic routes** with care; pre-generate if possible.
* Leverage **middleware** for auth, redirects, locale handling.

## State Management

* Local state → React `useState`, `useReducer`.
* Server state → **SWR** or **React Query**.
* Global state → Zustand, Redux Toolkit, or Context API (sparingly).

## Styling

* Use **TailwindCSS** or **CSS Modules**.
* For design systems, consider **shadcn/ui** or Chakra UI.
* Use **CSS variables** for theming (dark mode, branding).

## Performance

* Use **Image Optimization (`next/image`)**.
* Leverage **Font Optimization** via `next/font`.
* Prefetch routes with `next/link`.
* Use **Edge Middleware** for low-latency personalization.
* Apply **bundle analysis** with `next-bundle-analyzer`.

```bash
ANALYZE=true next build
```

## Security

* Always enable **HTTPS & CSP headers** (`next-secure-headers`).
* Sanitize user input (XSS prevention).
* Use **environment variables** (`process.env`) with `next/config`.
* Enforce **RBAC/ABAC** in API routes.

## Observability

* Integrate **Vercel Analytics / Google Analytics**.
* Use **OpenTelemetry** or Sentry for tracing & error tracking.
* Log structured JSON for serverless observability.

## Testing

* Unit tests → Jest + React Testing Library.
* Integration → Playwright/Cypress.
* Snapshot testing for critical layouts.
* Use **Mock Service Worker (MSW)** for API mocking.

## Deployment

* Deploy to **Vercel** (first-class support).
* For enterprise, deploy to **Azure Static Web Apps, AWS Amplify, or custom Docker**.
* Use **serverless DBs** (Planetscale, Supabase, CosmosDB) with Prisma/Drizzle.

## Anti-Patterns

* Using SSR everywhere (adds latency, cost).
* Over-fetching in `getServerSideProps`.
* Bloated `_app.tsx` with too much logic.
* Ignoring caching headers (`Cache-Control`).
* Putting secrets in client-side code.

## Checklist

* [ ] TypeScript configured
* [ ] ESLint + Prettier enforced
* [ ] Data fetching strategy chosen (SSG/SSR/ISR/CSR)
* [ ] SWR/React Query for client state
* [ ] Security headers & env vars configured
* [ ] Error monitoring enabled
* [ ] Tested with Lighthouse & Web Vitals
