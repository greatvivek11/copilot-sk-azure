# 🛠️ Technical Blueprint: Cloud-Native AI Hub

**A detailed guide for building the AI Knowledge Hub on Azure, using Dapr, Cosmos DB, and the Hugging Face Inference Router.**

---

### 📚 Key Facts & References

-   **Semantic Kernel**: Microsoft’s open-source SDK for AI orchestration. [GitHub](https://github.com/microsoft/semantic-kernel) | [Docs](https://learn.microsoft.com/en-us/semantic-kernel/overview/)
-   **Azure Container Apps**: Managed environment for running microservices and containerized applications. [Docs](https://learn.microsoft.com/en-us/azure/container-apps/)
-   **Dapr**: APIs for building resilient, stateful, and event-driven distributed applications. [Docs](https://docs.dapr.io/)
-   **Azure Cosmos DB for MongoDB (vCore)**: A powerful, scalable NoSQL database with integrated vector search capabilities. [Vector Search Docs](https://learn.microsoft.com/en-us/azure/cosmos-db/mongodb/vcore/vector-search)
-   **Hugging Face Router**: A proxy for routing requests to various open-source models. [Website](https://huggingface.co/inference-endpoints/router)

---

## 📋 A. Architecture & Technology Stack

- **Frontend**: React 19 + Vite 7 + Tailwind v4 + shadcn/ui.
- **Backend**: .NET 8/10 Minimal APIs, Vertical Slice Architecture, Mediator (source-gen)
- **Infra**: Azure Container Apps (FE & BE as separate container apps).
- **Service Communication**: Dapr service invocation.
- **Data**:
  - Azure SQL Database (relational data)
  - Azure Blob Storage (file storage)
  - Azure Cosmos DB (vector embeddings & memory)
- **AI Orchestration**:
  - Semantic Kernel
  - Hugging Face Inference Router via SK OpenAI connector
- **IaC**: Bicep/Terraform scripts to provision ACA, SQL, Blob, Cosmos, network.
- **Deployment**: GitHub Actions → GHCR → ACA.
* **Observability**: ACA logs, optional App Insights + Log Analytics (stay within free tier)

---

## B. Phased Implementation Plan

See: [Phased-Plan.md](/docs/plans/Phased_Plan.md).

---

## C. Architecture Principles

* **Vertical Slice**: Features implemented end-to-end (API → Domain → Infra).
* **Mediator**: Source-generated mediator pattern for clean command/query separation.
* **Dapr-first**: FE invokes BE via Dapr sidecar app-id.
* **Cloud-native**: Stateless services, externalized state in SQL/Blob/Cosmos.
* **Cost-aware**: Free-tier defaults, enterprise add-ons only documented.
* **Extensible**: AI provider abstracted (SK service layer) to swap providers easily.

---

## D. Repository Structure

```
/
 ├── frontend/     # React app
 ├── backend/      # .NET Minimal API, VSA slices
 ├── infra/        # Bicep/Terraform IaC
 ├── docs/         # Background, Phased-Plan, Blueprint, Architecture, Decisions
 └── .github/      # Workflows for CI/CD
```

---

## E. Success Criteria

* **Phase 0**: Repo, CI/CD, ACA deploy, Dapr health endpoint works.
* **Phase 1**: End-to-end chat, persisted sessions, streaming.
* **Phase 2**: RAG with documents, citations visible.
* **Phase 3–7**: Vision, insights, memory, agents, polish.
* Documentation and diagrams capture all design trade-offs.
