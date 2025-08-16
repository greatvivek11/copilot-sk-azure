# Project: AI-Powered Knowledge Hub (Cloud-Native Edition)

An enterprise-grade, AI-powered assistant built on a modern, scalable, and secure cloud architecture using Azure Container Apps, Dapr, Azure SQL, Azure Blob Storage, Azure Cosmos DB (vector search), Semantic Kernel, and the Hugging Face Inference Router (via SK OpenAI connector).

---
# Background

This project is designed as a **PoC + portfolio showcase** demonstrating modern enterprise AI application development with .NET, React, and Azure. The goal is to validate your skills across **C#/.NET 10, React 19, Azure Container Apps, Dapr, Semantic Kernel, and Hugging Face Inference API** while creating a functional product you can share.

---

## Vision

Build an **AI-Powered Knowledge Hub** combining conversational AI, document retrieval (RAG), sentiment analysis, vision/OCR, and insights dashboards. It will be production-like but scoped to free-tier friendly PoC deployments.

---

## Why This Project Matters

* **Enterprise relevance**: Azure-first deployment (ACA, SQL, Blob, Cosmos).
* **Modern stack**: .NET 10 Minimal APIs, React 19 + Vite + Tailwind v4, Dapr.
* **AI integration**: Semantic Kernel + Hugging Face Router via OpenAI connector.
* **Architecture**: Vertical Slice Architecture, Mediator (source-gen), repository pattern, IaC (Bicep/Terraform).
* **Showcase**: Demonstrates frontend, backend, AI/ML integration, cloud-native deployment, and CI/CD.

---

## Use Case

This application enables employees to:

- Chat with an AI assistant that searches and reasons over company documents.
- Upload documents/images (PDF, DOCX, TXT, images) to Azure Blob Storage.
- Ask grounded questions and get answers with citations from ingested documents.
- Analyze sentiment of customer feedback and internal notes.
- Retain long-term memory of conversations and user preferences.
- Perform image analysis and OCR for scanned/visual data.
- Use "Ask Me Anything" for general purpose queries.

---

## Key Features & Cloud Architecture

### Azure-first, Dapr-enabled

- Hosting: Azure Container Apps (ACA) hosts both frontend (React 19 + Vite + Tailwind v4) and backend (.NET Minimal APIs, Vertical Slice) as separate container apps.
- Service Invocation: Dapr sidecars provide secure FE -> BE invocation (FE -> Dapr -> BE), retries, mTLS, and service discovery.
- Data:
  - Azure SQL Database -> relational data (users, chat sessions, messages, audit).
  - Azure Blob Storage -> binary objects (documents, images).
  - Azure Cosmos DB (Consumption / MongoDB vCore with Vector Search) -> embeddings, RAG chunks, conversation memory vectors.
- Networking:
  - Custom domain for FE (public).
  - Private internal networking for BE + data planes; restrict access via Private Link / ACA internal ingress and Dapr service invocation only.
- AI Orchestration:
  - Semantic Kernel (SK) as coordinator.
  - Hugging Face Inference Router via SK OpenAI connector for chat, embeddings, and multimodal where available (model-agnostic, swappable).

---

## Enterprise Relevance

- Cloud-native microservices with ACA + Dapr.
- Secure data paths (private networking, least privilege, managed identities later).
- Real-world AI patterns: RAG, multimodal, memory, structured outputs, agents.
- Modern .NET & React best practices (Vertical Slice, Minimal APIs, Tailwind v4).

---

## Roadmap

See: [Phased-Plan.md](/docs/plans/Phased_Plan.md) for detailed roadmap and milestones.

---

## Constraints

* No personal Azure spend: use free tiers, consumption plans, and minimal SKUs.
* Secrets via GitHub Actions only (no Key Vault).
* No Private Endpoints or Defender for Cloud (documented but not implemented).
* Focus on **learning + showcasing**, not enterprise hardening.

---

## Outcomes

* Functional, cloud-hosted application accessible with custom domain (frontend).
* Backend internal to ACA, invoked via Dapr.
* Documentation of design decisions, trade-offs, and enterprise-ready recommendations.
* Resume-ready GitHub repo with clean code, docs, and architecture diagrams.
