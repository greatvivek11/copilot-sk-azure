# ☁️ Cloud-Native Architecture

This document provides the holistic, end-to-end architecture for deploying, securing, and operating the AI-Powered Knowledge Hub on Microsoft Azure. It connects the application design to the underlying cloud infrastructure, focusing on security, automation, observability, and cost-awareness. The architecture is designed to integrate cleanly with **.NET Aspire**, **ACA**, **Dapr**, and **Next.js SSR** frontend.

---

## 🎯 Core Principles

1. **Zero-Trust Security**: No component is trusted by default. All interactions, from user to frontend, frontend to backend, and backend to Azure services, must be authenticated and authorized.
2. **Infrastructure as Code (IaC)**: The entire cloud environment is defined in code, ensuring it is reproducible, version-controlled, and can be deployed automatically.
3. **Automation**: The entire lifecycle, from code commit to deployment, is automated via a CI/CD pipeline.
4. **Comprehensive Observability**: The system is designed to be observable, with structured logging, metrics, and tracing enabled across all components.
5. **Cost Awareness**: Optimize for free-tier and consumption-based pricing to keep PoC costs minimal while maintaining enterprise alignment.

---

## 📊 Overall Architecture Diagram

```mermaid
graph TD
    subgraph "User's Browser"
        A[Next.js SSR Client]
    end

    subgraph "CI/CD Pipeline"
        B(GitHub Actions) --> C{GitHub Container Registry (GHCR)};
    end

    subgraph "Azure"
        D[Microsoft Entra ID]
        
        subgraph "Azure Container App Environment (Internal Network)"
            E[Frontend ACA (Next.js SSR)] --> F(Dapr Sidecar);
            F --> G[Backend ACA (.NET Aspire + Minimal APIs)];
            G -- "Uses" --> H(User-Assigned Managed Identity);
        end

        subgraph "Azure Data & Secret Services"
            I[Azure SQL DB]
            J[Azure Blob Storage]
            K[Azure Cosmos DB (Vector)]
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

1. **Frontend Authentication**: User accesses the Next.js SSR app. The server-side uses **MSAL Node** to authenticate with **Microsoft Entra ID** and securely manage tokens.
2. **Token Acquisition**: After login, Entra ID issues **ID Token** and **Access Token**. SSR backend holds tokens securely, and client only receives session cookie.
3. **Dapr Service Invocation**: The Next.js server calls backend APIs via Dapr sidecar using service invocation. The access token is attached in headers.
4. **Backend Token Validation**: Backend validates JWT tokens against Entra ID, processing requests only if valid.

### 2. Secure Backend Integrations with Managed Identity

* **Identity Provider**: Backend ACA assigned a **User-Assigned Managed Identity**.
* **RBAC Roles**:

  * Azure SQL: `db_datareader`, `db_datawriter`
  * Azure Blob: `Storage Blob Data Contributor`
  * Azure Cosmos DB: data plane role for vector operations
* **Third-Party Secrets**: HF/OpenAI keys stored in **GitHub Actions secrets**, injected at deploy.
* **Configuration**: Backend uses `DefaultAzureCredential` in Aspire to authenticate seamlessly with Azure resources.

### 3. Infrastructure as Code (IaC) & CI/CD

* **IaC with Bicep**: Entire Azure environment (ACA, DBs, storage, identities) is defined in Bicep.
* **CI/CD with GitHub Actions**:

  1. Build & test frontend (Next.js) and backend (.NET Aspire).
  2. Build container images and push to GHCR.
  3. Deploy infra via Bicep.
  4. Run EF Core migrations against Azure SQL.
  5. Deploy ACA revisions with rollout strategies.

### 4. Observability & Monitoring

* **Centralized Logging**: All ACA logs to **Log Analytics Workspace**.
* **App Insights**: Integrated for both frontend (via SDK) and backend (via OTel from Aspire).
* **Distributed Tracing**: Correlation IDs flow from frontend to backend via Dapr.
* **Core Web Vitals**: Captured from frontend for UX monitoring.

---

## 🌐 Networking & Security

* **Ingress**: Only frontend ACA exposed publicly via custom domain + TLS.
* **Internal Only**: Backend ACA internal, accessible only via Dapr.
* **Secrets**: GH Actions secrets → injected into Aspire → ACA env vars.
* **Cost Controls**: Skipping Key Vault, Private Endpoints, Defender in PoC.

---

## 🔒 Enterprise-Grade Optional Enhancements

For production:

* Private Endpoints for all data services.
* Azure Key Vault for centralized secret management.
* Defender for Cloud for container security and compliance.

In PoC:

* Secure defaults with Managed Identity, HTTPS, and secret injection via CI/CD.

---

## ✅ Success Criteria

* ACA runs FE (Next.js SSR) and BE (.NET Aspire minimal APIs) with Dapr.
* Azure SQL, Blob, Cosmos vector all integrated.
* Aspire orchestrates local and cloud deployments consistently.
* Observability wired with free App Insights/Log Analytics.
* CI/CD fully automated with GHCR and GH Actions.
* Secure flows in place while staying within free-tier/consumption limits.
