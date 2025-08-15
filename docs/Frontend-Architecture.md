# 🖥️ Frontend Architecture

This document outlines the architectural decisions, technology stack, and design principles for the frontend of the AI-Powered Knowledge Hub. The goal is a clean, minimal, performant, and aesthetically pleasing user interface that showcases modern frontend development practices.

---

## 🎯 Core Principles

1.  **Performance First**: Given the scale-to-zero nature of Azure Container Apps, we must optimize for a fast First Contentful Paint (FCP) and a snappy user experience, even on a cold start.
2.  **Developer Experience**: The chosen tools should be modern, efficient, and enjoyable to work with, maximizing productivity.
3.  **Component Ownership**: We will favor tools that give us full control over the component code, allowing for easy customization and long-term maintainability.
4.  **Pragmatic State Management**: We will use the simplest, most effective state management solution that meets the project's needs without introducing unnecessary complexity.

---

## 🛠️ Technology Stack

This stack is chosen to be modern, performant, and well-suited for building a sophisticated single-page application.

| Category            | Technology                                | Justification                                                                                                                                                             |
| ------------------- | ----------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Runtime/Toolkit** | **Bun**                                   | All-in-one runtime, package manager, and bundler. Its exceptional speed is ideal for a fast development loop and efficient builds.                                        |
| **Framework**       | **React 19 + Vite**                       | Vite provides a best-in-class development server and optimized builds. React remains the standard for building rich, interactive UIs.                                     |
| **Rendering**       | **SPA (Single Page App)**                 | Optimized for static file hosting on ACA, minimizing cold start impact. Simpler to deploy and manage compared to SSR for this use case.                                   |
| **Styling**         | **Tailwind CSS v4 + shadcn/ui**           | The perfect combination for a modern, utility-first workflow. `shadcn/ui` provides beautifully designed, unstyled components that we can own and customize completely.      |
| **State (Global)**  | **Zustand**                               | A minimal, fast, and scalable state management solution. Perfect for managing shared state like user sessions or themes without the boilerplate of larger libraries.     |
| **State (Server)**  | **TanStack Query**                        | The industry standard for managing server state. It will handle all data fetching, caching, and synchronization with the backend, dramatically simplifying the UI logic. |
| **Language**        | **TypeScript**                            | Provides essential type safety, improving code quality and maintainability.                                                                                               |

---

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

1.  **Panel 1: Icon Sidebar (Far Left)**
    -   A thin, static vertical bar containing icons for the main application modules:
        -   💬 Chat
        -   📚 Documents
        -   📊 Insights
        -   ⚙️ Settings
2.  **Panel 2: Navigation / History Panel**
    -   The content of this panel is contextual, driven by the selection in the Icon Sidebar.
    -   If "Chat" is active, it lists previous chat sessions.
    -   If "Documents" is active, it shows a searchable file tree of ingested documents.
3.  **Panel 3: Main Content Area**
    -   This is the primary workspace where all user interaction takes place. It will render the active view, whether it's the chat window, a document reader, or the data visualization dashboard.

---

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
│   ├── pages/                  // Top-level page components
│   │
│   ├── services/               // Dapr client wrappers, auth services
│   │
│   └── stores/
│       └── useAuthStore.ts     // Zustand store for authentication state
│
├── .gitignore
├── bun.lockb
├── index.html
├── package.json
├── postcss.config.js
├── tailwind.config.js
└── tsconfig.json
```

### Key Architectural Decisions

-   **API Layer**: All interactions with the backend (via Dapr) will be encapsulated in custom hooks built with TanStack Query. UI components will be completely decoupled from the data-fetching logic.
-   **Error Handling**: We will implement a robust error handling strategy using error boundaries in React and centralized error logic in our API hooks.
-   **Authentication**: We will use the **Microsoft Authentication Library (MSAL) for React** (`@azure/msal-react`) to integrate with **Microsoft Entra ID**. A dedicated authentication service will wrap the MSAL logic, and the user's authentication state will be managed in a Zustand store.
---

## 🔧 Implementation & Operational Guidance

The following points provide implementation-level guidance and operational considerations that align with our core architectural principles.

### 1. Cold-Start & FCP Optimizations (Azure Container Apps)
- **Code Splitting**: Use route-based code splitting (`React.lazy` + dynamic imports) for feature panels to keep the initial JS bundle minimal.
- **Critical Path**: Ship only the application shell and critical UI for the first paint; lazy-load heavy components like document viewers or complex analytics.
- **Asset Caching**: Use HTTP/edge caching via a CDN for static assets. Employ `preconnect` and `preload` headers for critical resources like fonts.
- **Critical CSS**: Deliver critical CSS inlined for the application shell to improve FCP, and defer the loading of non-critical CSS.
- **Skeleton UIs**: Provide skeleton loaders and content placeholders to give immediate feedback while background API calls are in flight.

### 2. API Layer & TanStack Query Best Practices
- **Centralized Configuration**: Establish a central configuration for the TanStack Query client with sensible defaults for `staleTime`, `cacheTime`, and `retry` policies.
- **Global Error Handling**: Implement global error handling for network and authentication failures using TanStack Query's error handling mechanisms.
- **Optimistic Updates**: Use optimistic updates for user actions like sending a message or deleting a document to make the UI feel instantaneous.
- **Efficient Data Loading**: Use pagination and infinite queries for long lists, such as document lists and chat history.
- **API Client Wrapper**: Create a thin wrapper around the `fetch` API in `src/api/clients/` to handle request creation, auth header injection, and telemetry hooks.

### 3. State Management Patterns (Zustand + TanStack Query)
- **Clear Separation**: Strictly use TanStack Query for server-derived data and Zustand for ephemeral UI state (e.g., panel visibility, modal state, theme).
- **Derived State**: For state that bridges both, like a `selectedDocumentId`, store the ID in Zustand and use it to drive the corresponding TanStack Query for the document's content.
- **Persistence**: Use Zustand's persistence middleware to save non-sensitive UI state (like theme settings) to `localStorage`.

### 4. Authentication & Security
- **Authentication Flow**: We will use **MSAL for React** to implement the OAuth 2.0 Authorization Code Flow with PKCE. This is the recommended, most secure method for SPAs. MSAL will handle the complexities of token acquisition, silent renewal, and cacheing.
- **Token Management**: MSAL will manage the token lifecycle. The access token acquired for the backend API will be attached to outgoing requests in our API client wrapper. We will configure the backend to validate these tokens issued by Microsoft Entra ID.
- **Secure Headers**: Implement a strong Content Security Policy (CSP) and other security headers at the CDN or ACA ingress level.
- **Input Sanitization**: Validate and sanitize all user-provided content, especially for uploaded documents that might be rendered, to prevent XSS attacks.

### 5. Observability & Performance Monitoring
- **Core Web Vitals**: Integrate the `web-vitals` library to capture FCP, LCP, CLS, etc., and send the data to a telemetry sink like Azure Application Insights.
- **API Telemetry**: Instrument TanStack Query events to monitor cache hit rates, query failures, and API performance from the client's perspective.
- **Error Tracking**: Use a service like Sentry or Application Insights to capture and report user-facing errors, correlating them with backend trace IDs.

### 6. Offline Support & Resilience
- **Service Worker**: Implement a basic Service Worker to cache the application shell and core assets for faster subsequent loads and basic offline capability.
- **Data Caching**: For a more advanced offline experience, cache recently viewed documents in `IndexedDB` (using a library like `idb`).
- **Network Status**: Use a hook like `useNetworkStatus` to detect online/offline state and adjust UI/functionality accordingly (e.g., disable upload buttons when offline).

### 7. Testing & CI/CD
- **Testing Pyramid**:
  - **Unit/Component**: Use Vitest and React Testing Library for components, hooks, and utility functions.
  - **E2E**: Use Playwright to test critical user flows like authentication, chat, and document upload.
- **Performance Budgeting**: Integrate Lighthouse checks or Web Vitals thresholds into the CI pipeline to prevent performance regressions.
- **CI/CD Pipeline**: Use GitHub Actions to build the application (with Bun or in a Docker container), run all tests, and deploy the final artifacts to Azure Container Apps.

### 8. Accessibility (A11y) & UX
- **A11y as a Requirement**: Make accessibility a core part of the "definition of done" for all UI components. This includes keyboard navigation, semantic HTML, and proper ARIA attributes.
- **Automated Checks**: Use `axe-core` in the CI pipeline to automatically catch accessibility regressions.
- **Perceived Performance**: Use skeleton loaders, progressive disclosure of UI elements, and transition animations to make the application feel faster.