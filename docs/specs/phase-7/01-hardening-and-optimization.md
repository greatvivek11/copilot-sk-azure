# Phase 7 Spec: Hardening & Optimization

**Component:** Full Stack (Platform, Backend, Frontend)  
**Source Plan:** [`docs/plans/Phased_Plan.md`](/docs/plans/Phased_Plan.md)

## 1. Goal

To apply production-ready polish to the application. This involves implementing robust security, optimizing for performance and cost, and ensuring the project is thoroughly documented and ready to be showcased.

## 2. Feature Breakdown & Acceptance Criteria

### 2.1. Security Hardening

-   **Goal:** To secure the application from common threats and ensure access is properly controlled.
-   **Acceptance Criteria:**
    -   [ ] **Authentication with Entra ID:**
        -   [ ] An Azure App Registration is created for the application in Microsoft Entra ID.
        -   [ ] The frontend is configured to use the Microsoft Authentication Library (MSAL) to handle the OAuth 2.0 login flow with Entra ID.
        -   [ ] The backend API is configured to validate the JWTs issued by Entra ID. All endpoints (except the public `/health` and `/ping-ai` checks) are secured and require a valid token.
    -   [ ] **Authorization with Managed Identity:**
        -   [ ] The backend application's User-Assigned Managed Identity is used to connect to Azure SQL, Blob Storage, and Cosmos DB.
        -   [ ] Connection strings containing secrets are completely removed from the application configuration and GitHub Actions secrets.
        -   [ ] The Managed Identity is granted the most restrictive, least-privilege RBAC roles required for each service (e.g., `Storage Blob Data Reader`, `AcrPull`, etc.).
    -   [ ] **PII Redaction (Conceptual):**
        -   [ ] A dedicated `PII-Redaction` native function or middleware is created.
        -   [ ] This function is applied to all logs to ensure sensitive user data is not persisted in plain text.

### 2.2. Performance & Cost Optimization

-   **Goal:** To ensure the application runs efficiently and stays within the free-tier constraints.
-   **Acceptance Criteria:**
    -   [ ] **RAG Optimization:**
        -   [ ] The document chunking strategy is reviewed and tuned (e.g., adjusting chunk size and overlap) to provide the best balance of context quality and token count.
    -   [ ] **Caching:**
        -   [ ] A caching layer (e.g., using an in-memory cache for simplicity, or a Dapr state store component pointing to Redis) is implemented to cache frequently requested data, such as AI model responses for identical prompts.
    -   [ ] **Cost Controls:**
        -   [ ] The application is configured to control costs by default (e.g., limiting the `max_tokens` in AI requests, using smaller/cheaper models for simpler tasks like summarization).
    -   [ ] **Model Selection:**
        -   [ ] The system is configured to use different AI models for different tasks based on a cost/performance analysis (e.g., a smaller, faster model for summarization; a larger, more powerful model for complex agent tasks).
    -   [ ] **Autoscaling Rules:**
        -   [ ] The autoscaling rules for the Azure Container Apps are fine-tuned based on observed traffic patterns to optimize for both responsiveness and cost (e.g., adjusting CPU/memory thresholds, concurrency settings).

### 2.3. Final Documentation & Demo Script

-   **Goal:** To make the project easily understandable, maintainable, and presentable.
-   **Acceptance Criteria:**
    -   [ ] The root `README.md` is updated to be a comprehensive guide, covering the project's purpose, architecture, and detailed setup instructions.
    -   [ ] All architectural diagrams in `/docs` are reviewed and updated to match the final implementation.
    -   [ ] A detailed, step-by-step `DEMO_SCRIPT.md` is created, outlining how to showcase all key features of the application from Phase 0 to Phase 7.

## 3. Definition of Done (DoD)

-   [ ] A user must log in via a Microsoft Entra ID account to access the application.
-   [ ] The application connects to all Azure data services using Managed Identity (no secrets in config).
-   [ ] The project's cost profile is documented, and guardrails are in place to stay within free-tier limits.
-   [ ] The repository is clean, well-documented, and ready to be shared as a portfolio piece.