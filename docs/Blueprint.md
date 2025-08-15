# 🛠️ Technical Blueprint: Cloud-Native AI Hub

**A detailed guide for building the AI Knowledge Hub on Azure, using Dapr, Cosmos DB, and the Hugging Face Inference Router.**

---

## 📋 A. Architecture & Technology Stack

- **Frontend**: React 19 + Vite 7 + Tailwind v4 + shadcn/ui.
- **Backend**: .NET 8/10 Minimal APIs, Vertical Slice Architecture, MediatR.
- **Hosting**: Azure Container Apps (FE & BE as separate container apps).
- **Service Communication**: Dapr service invocation.
- **Data**:
  - Azure SQL Database (relational data)
  - Azure Blob Storage (file storage)
  - Azure Cosmos DB (vector embeddings & memory)
- **AI Orchestration**:
  - Semantic Kernel
  - Hugging Face Inference Router via SK OpenAI connector
- **IaC**: Bicep/Terraform scripts to provision ACA, SQL, Blob, Cosmos, network.
- **CI/CD**: GitHub Actions → ACR → ACA.

### 📚 Key Facts & References

-   **Semantic Kernel**: Microsoft’s open-source SDK for AI orchestration. [GitHub](https://github.com/microsoft/semantic-kernel) | [Docs](https://learn.microsoft.com/en-us/semantic-kernel/overview/)
-   **Azure Container Apps**: Managed environment for running microservices and containerized applications. [Docs](https://learn.microsoft.com/en-us/azure/container-apps/)
-   **Dapr**: APIs for building resilient, stateful, and event-driven distributed applications. [Docs](https://docs.dapr.io/)
-   **Azure Cosmos DB for MongoDB (vCore)**: A powerful, scalable NoSQL database with integrated vector search capabilities. [Vector Search Docs](https://learn.microsoft.com/en-us/azure/cosmos-db/mongodb/vcore/vector-search)
-   **Hugging Face Router**: A proxy for routing requests to various open-source models. [Website](https://huggingface.co/inference-endpoints/router)

---

## 🗺️ B. Phased Implementation Plan

### Phase 0 — Cloud Foundation
- Containerize FE/BE.
- Provision infra via IaC.
- Deploy to ACA with Dapr enabled.
- `/api/health` reachable via Dapr from FE.

### Phase 1 — Core Chat
- Backend SK + HF chat completion.
- FE chat UI with session persistence.
- Save chat to SQL.

### Phase 2 — Document RAG
- Upload docs to Blob → extract text → embed in Cosmos.
- Retrieval pipeline for doc Q&A.

### Phase 3 — Vision
- Image upload + multimodal analysis.
- OCR pipeline for text extraction.

### Phase 4 — Sentiment
- Bulk sentiment endpoint.
- FE insights dashboard.

### Phase 5 — Memory
- Summarize/store past chats in Cosmos vectors.
- Personalize responses.

### Phase 6 — Agents
- SK tools/plugins.
- Orchestrated multi-step reasoning.

### Phase 7 — Hardening
- AuthN/AuthZ.
- Cost controls & monitoring.
- Documentation & demo polish.

---

## 📦 C. Tech Stack & NuGet Packages

### 🛠️ Base Tools & Services

-   .NET 8 SDK
-   ASP.NET Core Web API (using Vertical Slice Architecture)
-   Azure CLI / PowerShell
-   Docker
-   Dapr CLI

### 🧩 Core .NET Packages

```bash
# Semantic Kernel core
dotnet add package Microsoft.SemanticKernel

# OpenAI connector (to connect to Hugging Face Router)
dotnet add package Microsoft.SemanticKernel.Connectors.OpenAI

# Entity Framework Core for Azure SQL
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Design

# Azure SDKs
dotnet add package Azure.Identity
dotnet add package Azure.Storage.Blobs
dotnet add package Azure.AI.OpenAI # For official Azure OpenAI services if needed later

# Cosmos DB & MongoDB Driver
dotnet add package MongoDB.Driver

# MediatR for VSA
dotnet add package MediatR
```

---

## 💻 D. Code Snippets & Configuration

### Connecting to Hugging Face Router

Here’s how to configure the `Kernel` to use the Hugging Face Router via the OpenAI connector. The `hfToken` should be stored securely in Azure Key Vault and accessed via Managed Identity.

```csharp
using Microsoft.SemanticKernel;

var kernelBuilder = Kernel.CreateBuilder();
var hfToken = //... load from Azure Key Vault

// Add the OpenAI Chat Completion connector, pointing to the HF Router endpoint
kernelBuilder.AddOpenAIChatCompletion(
    modelId: "your-chosen-model-id", // e.g., "meta-llama/Llama-2-7b-chat-hf"
    apiKey: hfToken,
    endpoint: new Uri("https://api-inference.huggingface.co") // Or your dedicated endpoint
);

var kernel = kernelBuilder.Build();
```

### Dapr Service Invocation (Frontend to Backend)

In your React app, instead of calling the backend URL directly, you'll invoke it through the Dapr sidecar.

```typescript
// Example in a React component
async function callApi() {
  // Dapr sidecar runs on localhost:3500 by default
  const daprPort = process.env.DAPR_HTTP_PORT || 3500;
  
  // 'aihub-backend' is the Dapr App ID of your backend service
  const response = await fetch(`http://localhost:${daprPort}/v1.0/invoke/aihub-backend/method/api/chat`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ userId: 'user-1', message: 'Hello from frontend!' })
  });

  const data = await response.json();
  console.log(data);
}
```

---

##  G. Suggested Repository Structure

The structure remains similar, but with the addition of infrastructure-as-code.

```
/
├── .github/                     # CI/CD workflows
├── infra/                       # Bicep/Terraform scripts
├── src/
│   ├── AIHub.Backend/         # The single backend project
│   └── AIHub.Frontend/        # The React frontend project
└── .gitignore