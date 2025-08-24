# 🖥️ Frontend Architecture

This document outlines the architectural decisions, technology stack, and design principles for the frontend of the AI-Powered Knowledge Hub. The goal is a clean, minimal, performant, and aesthetically pleasing user interface that showcases modern frontend development practices.

## 🎯 Core Principles

1. **Performance First**: Given the scale-to-zero nature of Azure Container Apps, we must optimize for a fast First Contentful Paint (FCP) and a snappy user experience, even on a cold start.
2. **Developer Experience**: The chosen tools should be modern, efficient, and enjoyable to work with, maximizing productivity.
3. **Component Ownership**: We will favor tools that give us full control over the component code, allowing for easy customization and long-term maintainability.
4. **Pragmatic State Management**: We will use the simplest, most effective state management solution that meets the project's needs without introducing unnecessary complexity.
5. **Server-Side Rendering & BFF**: Shift from pure SPA to **Next.js SSR** with **Backend-for-Frontend (BFF)** route handlers. Secrets/config are kept server-side, improving security and initial load performance.
6. **Streaming-first**: Support real-time token streaming from the backend to the chat UI.

## 🛠️ Technology Stack

This stack is chosen to be modern, performant, and well-suited for building a sophisticated web application.

| Category            | Technology                      | Justification                                                                                                                                                          |
| ------------------- | ------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Runtime/Toolkit** | **Bun / Node.js**               | Bun remains preferred for local dev speed, but Next.js requires Node.js for SSR in production. Both runtimes are supported in ACA.                                     |
| **Framework**       | **React 19 + Next.js**          | Next.js enables SSR, streaming, and BFF route handlers. React 19 remains the core library for building interactive UIs.                                                |
| **Rendering**       | **SSR (Server-Side Rendering)** | SSR improves FCP, SEO, and hides secrets on the server. Route Handlers proxy requests to backend APIs via Dapr.                                                        |
| **Styling**         | **Tailwind CSS v4 + shadcn/ui** | The perfect combination for a modern, utility-first workflow. `shadcn/ui` provides beautifully designed, unstyled components that we can own and customize completely. |
| **State (Global)**  | **Zustand**                     | A minimal, fast, and scalable state management solution. Perfect for managing shared state like UI themes or ephemeral state.                                          |
| **State (Server)**  | **TanStack Query**              | Manages server data fetching, caching, and synchronization. In SSR, queries can be pre-fetched on the server for faster hydration.                                     |
| **Language**        | **TypeScript**                  | Provides essential type safety, improving code quality and maintainability.                                                                                            |

## 🎨 UI/UX Vision: A Three-Panel Layout

To achieve a clean, intuitive, and scalable interface, we will adopt a three-panel layout common in modern desktop and web applications.

```
+------+-------------------------+-----------------------------------------+
|      |                         |                                         |
| Icon |   Navigation / History  |           Main Content Area             |
| Side |       (Panel 2)         |                (Panel 3)                |
| Bar  |                         |                                         |
| (P1) | - Chat History          | - Chat Interface                        |
|      | - Document Explorer     | - Document Viewer                       |
|      | - ...                   | - Insights Dashboard                    |
|      |                         |                                         |
+------+-------------------------+-----------------------------------------+
```

1. **Panel 1: Icon Sidebar (Far Left)**

   * A thin, static vertical bar containing icons for the main application modules:

     * 💬 Chat
     * 📚 Documents
     * 📊 Insights
     * ⚙️ Settings
2. **Panel 2: Navigation / History Panel**

   * The content of this panel is contextual, driven by the selection in the Icon Sidebar.
   * If "Chat" is active, it lists previous chat sessions.
   * If "Documents" is active, it shows a searchable file tree of ingested documents.
3. **Panel 3: Main Content Area**

   * This is the primary workspace where all user interaction takes place. It will render the active view, whether it's the chat window, a document reader, or the data visualization dashboard.

## 📁 Project Structure

The frontend project will be organized by feature, promoting modularity and separation of concerns.

```
/src/AIHub.Frontend/
|
├── public/                     // Static assets
├── src/
│   ├── api/                    // API hooks using TanStack Query
│   │   └── chat.ts             // e.g., useGetChatHistory(), useSendMessage()
│   │
│   ├── assets/                 // Images, fonts, etc.
│   │
│   ├── components/
│   │   ├── layout/             // Main layout components (Sidebar, Panels, etc.)
│   │   └── ui/                 // Components from shadcn/ui (Button, Input, etc.)
│   │
│   ├── features/
│   │   ├── chat/               // All components related to the chat feature
│   │   └── documents/          // All components related to document management
│   │
│   ├── hooks/                  // Custom React hooks
│   │
│   ├── lib/                    // Utility functions, constants
│   │
│   ├── pages/                  // Top-level page components (SSR pages)
│   │
│   ├── services/               // Dapr client wrappers, auth services
│   │
│   └── stores/
│       └── useAuthStore.ts     // Zustand store for authentication state
|
├── .gitignore
├── bun.lockb
├── next.config.js              // Next.js config (SSR, streaming)
├── package.json
├── postcss.config.js
├── tailwind.config.js
└── tsconfig.json
```

### Key Architectural Decisions

* **API Layer**: All interactions with the backend (via Dapr) will be encapsulated in custom hooks or Next.js route handlers. TanStack Query hydrates data on the client. UI components remain decoupled from fetch logic.
* **Error Handling**: Robust error handling with React error boundaries and centralized API hook logic.
* **Authentication**: With SSR, **MSAL for Node.js/NextAuth.js** can be integrated to manage Entra ID tokens on the server. Tokens are never exposed to the browser.
* **Streaming**: Chat endpoints stream responses progressively to improve UX.

## 🔧 Implementation & Operational Guidance

### 1. Cold-Start & FCP Optimizations (Azure Container Apps)

* **SSR Advantage**: First request served server-side, reducing bundle size and improving FCP.
* **Code Splitting**: Still leverage dynamic imports for non-critical components.
* **Critical Path**: Stream HTML progressively with Next.js streaming SSR.
* **Asset Caching**: Use CDN edge caching for static assets.
* **Skeleton UIs**: Provide skeleton loaders to mask latency.

### 2. API Layer & TanStack Query Best Practices

* **Centralized Configuration**: TanStack Query client configured for SSR prefetch + client hydration.
* **Error Handling**: Global error handling for network/auth failures.
* **Optimistic Updates**: Still supported for chat/doc interactions.
* **API Client Wrapper**: Thin wrapper around `fetch` or Next.js `fetch` with Dapr routing.

### 3. State Management Patterns (Zustand + TanStack Query)

* **Clear Separation**: Zustand for ephemeral state, TanStack Query for server state.
* **Hydration**: Pre-fetched queries hydrated into client cache at SSR time.

### 4. Authentication & Security

* **Server-Side Auth**: Use Next.js route handlers with MSAL/NextAuth.js to acquire and cache Entra ID tokens server-side.
* **Token Management**: Tokens are attached to requests internally; browser never sees raw tokens.
* **Secure Headers**: CSP and other headers applied via Next.js middleware/ACA ingress.

### 5. Observability & Performance Monitoring

* **Web Vitals**: Capture metrics via `web-vitals` and forward to App Insights.
* **Error Tracking**: Integrate Sentry or App Insights.

### 6. Offline Support & Resilience

* **Service Worker**: Cache shell assets.
* **Data Caching**: IndexedDB optional for offline docs.
* **Network Awareness**: Detect and adapt to offline state.

### 7. Testing & CI/CD

* **Testing Pyramid**: Vitest + React Testing Library, Playwright for E2E.
* **SSR Testing**: Validate SSR-rendered output.
* **CI/CD**: GitHub Actions build → Dockerize → deploy to ACA.

### 8. Accessibility (A11y) & UX

* **Accessibility**: Semantic HTML, ARIA attributes, keyboard nav.
* **Automated Checks**: axe-core in CI.
* **Perceived Performance**: Skeletons + streaming + transitions.
