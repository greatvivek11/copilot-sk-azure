# ☁️ Cloud-Native Architecture

This document provides the holistic, end-to-end architecture for deploying, securing, and operating the AI-Powered Knowledge Hub on Microsoft Azure. It connects the application design to the underlying cloud infrastructure, focusing on security, automation, and observability.

---

## 🎯 Core Principles

1.  **Zero-Trust Security**: No component is trusted by default. All interactions, from user to frontend, frontend to backend, and backend to Azure services, must be authenticated and authorized.
2.  **Infrastructure as Code (IaC)**: The entire cloud environment is defined in code, ensuring it is reproducible, version-controlled, and can be deployed automatically.
3.  **Automation**: The entire lifecycle, from code commit to deployment, is automated via a CI/CD pipeline.
4.  **Comprehensive Observability**: The system is designed to be observable, with structured logging, metrics, and tracing enabled across all components.

---

##  diagrama Overall Architecture

```mermaid
graph TD
    subgraph "User's Browser"
        A[React SPA]
    end

    subgraph "CI/CD Pipeline"
        B(GitHub Actions) --> C{GitHub Container Registry (GHCR)};
    end

    subgraph "Azure"
        D[Microsoft Entra ID]
        
        subgraph "Azure Container App Environment (Internal Network)"
            E[Frontend Container App] --> F(Dapr Sidecar);
            F --> G[Backend Container App];
            G -- "Uses" --> H(User-Assigned Managed Identity);
        end

        subgraph "Azure Data & Secret Services"
            I[Azure SQL DB]
            J[Azure Blob Storage]
            K[Azure Cosmos DB]
            L(3rd Party Secrets <br/>in GitHub Actions)
        end
        
        M[Log Analytics Workspace]
        N[Application Insights]
    end

    A -- "1. Login (MSAL)" --> D;
    D -- "2. ID & Access Tokens" --> A;
    A -- "3. API Call w/ Token via Dapr" --> F;
    
    C --> G;
    C --> E;

    H -- "RBAC" --> I;
    H -- "RBAC" --> J;
    H -- "RBAC" --> K;
    B -- "Injects" --> G;
    
    G -- "Logs & Metrics" --> N;
    E -- "Logs & Metrics" --> N;
    N -- "Sends Data To" --> M;
```

---

## Component Breakdown & Flows

### 1. Authentication & Service Communication Flow

This flow ensures that only authenticated users can access the backend API.

1.  **Frontend Authentication**: A user accesses the React SPA. The app uses the **MSAL for React** library to redirect the user to the **Microsoft Entra ID** login page.
2.  **Token Acquisition**: After a successful login, Entra ID returns an **ID Token** (for user info) and an **Access Token** to the React app. The access token's audience (`aud` claim) is configured to be the backend API.
3.  **Dapr Service Invocation**: The React app makes an API call to its local Dapr sidecar endpoint (e.g., `http://localhost:3500/v1.0/invoke/...`). It includes the acquired access token in the `Authorization: Bearer <token>` header.
4.  **Secure Forwarding**: The Dapr sidecar forwards the request to the backend container app over the private network. Crucially, Dapr passes the `Authorization` header through to the backend.
5.  **Backend Token Validation**: The ASP.NET Core backend receives the request. Its authentication middleware (configured for JWTs from Entra ID) validates the token's signature, issuer, and audience. If the token is valid, the request is processed; otherwise, it is rejected with a `401 Unauthorized`.

### 2. Secure Backend Integrations with Managed Identity

The backend application will **never** handle secrets like connection strings for Azure resources.

-   **Identity Provider**: We will create a single **User-Assigned Managed Identity** in Azure. This identity acts as the application's security principal in the cloud.
-   **RBAC Roles**: This Managed Identity will be granted the least-privilege RBAC (Role-Based Access Control) roles required on each data service:
    -   **Azure SQL**: `db_datareader`, `db_datawriter`
    -   **Azure Blob Storage**: `Storage Blob Data Contributor`
    -   **Azure Cosmos DB**: A built-in or custom data plane role.
-   **Third-Party Secrets**: For secrets that are not part of Azure (like the Hugging Face API token), we will use **GitHub Actions secrets**. These will be securely injected into the backend container as environment variables only during the CI/CD deployment process.
-   **Application Configuration**: The backend container app will be assigned this Managed Identity. The application code will use the `DefaultAzureCredential` class from the Azure Identity SDK. This class automatically detects and uses the assigned Managed Identity when running in Azure, making the code free of any secrets. For local development, it can be configured to use a developer's logged-in Azure CLI credentials.

### 3. Infrastructure as Code (IaC) & CI/CD

-   **IaC with Bicep**: The entire Azure environment will be defined in `.bicep` files stored in the repository. This includes the Container App Environment, the container apps, all data services, the Managed Identity, and all RBAC role assignments. Bicep outputs will be used to pass resource names and IDs to subsequent pipeline steps, avoiding hardcoding.
-   **CI/CD with GitHub Actions**: A workflow in `.github/workflows/` will automate the entire deployment process:
    1.  **Trigger**: On push to the `main` branch.
    2.  **Build & Test**: Build the .NET and React applications, run all automated tests.
    3.  **Containerize**: Build Docker images for the frontend and backend.
    4.  **Push to Registry**: Push the images to the private **GitHub Container Registry (GHCR)** with version tags.
    5.  **Deploy Infrastructure**: Run the Bicep deployment to create or update the Azure resources.
    6.  **Database Migration**: As a preliminary step to deployment, the pipeline will run **Entity Framework Core migrations** against the Azure SQL database to apply any schema changes. This is often done using a containerized job or a CLI script that runs before the main application deployment.
    7.  **Deploy Application**: Trigger a new revision of the Azure Container Apps, pointing them to the newly pushed container images in GHCR. This enables safe deployment strategies like blue-green or percentage-based traffic splitting for testing new revisions before a full rollout.
-   **Scalability**: The Bicep templates will include auto-scaling rules for the Azure Container Apps (based on HTTP concurrency, CPU, or memory) to handle variable loads.

### 4. Observability & Monitoring

-   **Centralized Logging**: All services, including the frontend and backend container apps, will be configured to send logs and metrics to a central **Log Analytics Workspace**.
-   **Application Performance Monitoring (APM)**: **Application Insights** will be enabled for both the frontend (via its JS SDK) and backend (via its .NET SDK). This will provide:
    -   **Log Correlation**: The frontend will generate a correlation ID (`x-correlation-id`) for each user-initiated operation. This ID will be passed through Dapr to the backend and included in all logs and downstream API calls, enabling full end-to-end distributed tracing.
    -   Performance metrics (request rates, duration, failure rates).
    -   Client-side performance monitoring (Core Web Vitals).
    -   Exception and error tracking.
---

## 🔒 Enterprise-Grade Optional Enhancements

**Note**: The following are not implemented in the PoC due to cost considerations but are recommended for a production/enterprise deployment.

-   **Private Endpoints**: Using Private Endpoints for Azure SQL, Blob Storage, and Cosmos DB would provide complete network isolation from the public internet.
-   **Azure Key Vault**: For a production system, all secrets (including third-party API keys) should be managed centrally in Azure Key Vault with automated rotation policies.
-   **Microsoft Defender for Cloud**: This service provides advanced security posture management, including container image vulnerability scanning, threat detection, and compliance recommendations.

### PoC Security Approach

In this PoC, a strong security posture is maintained through:

-   **Internal Ingress**: The backend API is only accessible from within the private Azure Container App environment.
-   **Managed Identity**: All communication between the backend and other Azure resources is authenticated via a passwordless Managed Identity.
-   **GitHub Actions Secrets**: Third-party API keys are stored as encrypted secrets in GitHub and only injected during the CI/CD pipeline run.
-   **HTTPS**: All public-facing endpoints are secured with HTTPS.