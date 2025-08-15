# 🌟 Project: AI-Powered Knowledge Hub (Cloud-Native Edition)

**An enterprise-grade, AI-powered assistant built on a modern, scalable, and secure cloud architecture using Azure, Dapr, and Semantic Kernel.**

---

## 🎯 Use Case

This project aims to build a highly scalable web application where employees can:

-   💬 **Chat with an AI assistant** that searches and reasons over company documents.
-   📤 **Upload various file types** (PDFs, DOCX, TXT, images) to **Azure Blob Storage**.
-   ❓ **Ask questions** and receive grounded answers with citations from the source documents.
-   😊 **Analyze sentiment** of customer feedback or other text data.
-   🤔 Use an **"Ask Me Anything"** mode for general-purpose queries.

---

## ✨ Key Features & Architecture

### ☁️ Cloud-Native Architecture on Azure

The entire solution is designed to run on Azure, leveraging modern cloud services for scalability, security, and reliability.

-   **Hosting**: **Azure Container Apps** will host our frontend and backend applications as containerized services.
-   **Service Communication**: **Dapr (Distributed Application Runtime)** will be used for secure and reliable inter-service communication (e.g., Frontend → Backend).
-   **Data Storage**:
    -   **Azure SQL Database**: Serves as the primary relational database for structured data.
    -   **Azure Blob Storage**: For storing and managing all uploaded documents and images.
    -   **Azure Cosmos DB (Consumption Plan)**: Acts as our powerful and scalable vector database for RAG.
-   **Networking**: The frontend will be exposed to the public via a **custom domain**, while all backend services will remain on a private, internal network for enhanced security.

### 🖥️ Frontend (React)

-   🔐 **Secure Login**: Integration with an auth provider like Azure AD.
-   💬 **Chat Interface**: Modern, with support for streaming AI responses.
-   📤 **File Uploader**: Uploads files directly to a secure backend endpoint that routes them to Azure Blob Storage.
-   📊 **Insights Panel**: Displays sentiment breakdown of customer reviews.

### ⚙️ Backend (ASP.NET Core + Semantic Kernel)

-   🤖 **Chat Endpoint**: Powered by Semantic Kernel, using the **Hugging Face Inference Router** via the OpenAI connector for model flexibility.
-   👁️ **Vision/OCR**: Capabilities for analyzing images and extracting text.
-   😊 **Sentiment Analysis**: Implemented via carefully crafted prompts.
-   🧠 **Long-Term Memory**: Stores conversation summaries and document embeddings in Azure Cosmos DB.

---

## 🏢 Enterprise Relevance

-   **Cloud-Native Expertise**: Showcases skills in building and deploying applications on a modern Azure stack.
-   **Microservices & Dapr**: Demonstrates understanding of distributed application architecture and best practices.
-   **Scalable AI**: Highlights the ability to design and implement a scalable RAG and AI orchestration solution for enterprise use cases.

---

## 🚀 Phased Implementation Roadmap

> Each phase is a working milestone that builds towards the final feature set.

### Phase 0 — 🏗 Cloud Foundation & Setup
- Deploy containerized FE + BE to Azure Container Apps via GitHub Actions.
- Configure Dapr sidecars and service invocation.
- Set up Azure SQL, Blob, Cosmos DB (Consumption).
- Connect SK to Hugging Face Router (OpenAI connector).
- Deliverable: `/api/health` callable from FE via Dapr.

### Phase 1 — 💬 Core Chat Assistant
- Minimal APIs with Semantic Kernel + HF Router.
- FE chat UI with streaming responses.
- Store conversations/messages in Azure SQL.

### Phase 2 — 📚 Document Ingestion & RAG
- Upload docs → Blob Storage → extract text → embeddings in Cosmos DB.
- Backend RAG pipeline using Cosmos vector search.
- FE “Ask about docs” mode with citations.

### Phase 3 — 🖼 Vision & OCR
- Image ingestion to Blob + AI analysis via HF model.
- OCR pipeline for scanned PDFs/images → index in Cosmos.

### Phase 4 — 📊 Sentiment & Insights
- Batch sentiment analysis endpoint.
- FE dashboard: sentiment breakdown, trending topics.

### Phase 5 — 🧠 Long-Term Memory
- Store conversation summaries in Cosmos vectors.
- Load relevant memory per returning user.

### Phase 6 — 🛠 Agents & Tooling
- Add SK planner + tool plugins (e.g., internal API call, CSV export).
- AI chooses when to invoke tools.

### Phase 7 — 🔒 Hardening & Optimization
- Security hardening (AAD auth).
- Performance tuning & cost monitoring.
- Detailed README + architecture docs + live demo.