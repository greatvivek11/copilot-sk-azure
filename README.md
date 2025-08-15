# 🤖 AI-Powered Knowledge Hub

This repository contains the source code for a modern, cloud-native AI assistant designed to serve as an internal "Copilot" for enterprise knowledge. It allows users to chat with, upload, and analyze company documents using a sophisticated, scalable, and secure architecture.

This project is built to showcase advanced skills in full-stack development, AI integration, and cloud-native architecture on Microsoft Azure.

---

## ✨ Core Features

-   **Conversational AI**: Chat with an AI assistant that can reason over and answer questions about your documents.
-   **Document Ingestion**: Upload PDF, DOCX, TXT, and image files for the AI to process and index.
-   **Retrieval-Augmented Generation (RAG)**: Get answers grounded in your documents, complete with citations.
-   **Vision & OCR**: Analyze images and extract text from scanned documents.
-   **Sentiment Analysis**: Understand the sentiment of customer feedback or other text data.
-   **Secure & Scalable**: Built on a secure, cloud-native foundation using modern best practices.

---

## 🏛️ Architecture Overview

This project is built using a modern, distributed architecture designed for scalability, maintainability, and security.

-   **Backend**: A **.NET 8/10** application built with **ASP.NET Core Minimal APIs** following a **Vertical Slice Architecture (VSA)** with **MediatR** for clean, feature-focused code.
-   **Frontend**: A **React 19** Single Page Application (SPA) built with **Vite** and **Bun**, styled with **Tailwind CSS** and **shadcn/ui**.
-   **Cloud Platform**: Hosted entirely on **Azure Container Apps**, with a containerized frontend and backend.
-   **Service Communication**: **Dapr (Distributed Application Runtime)** is used for secure, internal service-to-service communication.
-   **AI Orchestration**: **Semantic Kernel** is used to orchestrate calls to the **Hugging Face Inference Router**, providing flexibility in model choice.
-   **Data Storage**:
    -   **Azure SQL Database**: For structured, relational data.
    -   **Azure Blob Storage**: For all uploaded documents and files.
    -   **Azure Cosmos DB**: For vector embeddings to power the RAG pipeline.
-   **Security**: Authentication is handled by **Microsoft Entra ID** using the **MSAL** library. All communication between the backend and Azure services uses passwordless **Managed Identities**.
-   **DevOps**: Infrastructure is defined with **Bicep (IaC)** and deployed automatically via a **GitHub Actions** CI/CD pipeline.

For a deeper dive, please see the detailed architectural documents:

-   **[Cloud Architecture](./docs/Cloud-Architecture.md)**: The holistic, end-to-end deployment and security plan.
-   **[Backend Architecture](./docs/Backend-Architecture.md)**: The internal structure of the .NET backend.
-   **[Frontend Architecture](./docs/Frontend-Architecture.md)**: The technology stack and patterns for the React frontend.

---

## 🚀 Getting Started

To run this project locally, you will need the following prerequisites installed:

-   [.NET 8 SDK](https://dotnet.microsoft.com/en-us/download/dotnet/8.0)
-   [Bun](https://bun.sh/)
-   [Docker Desktop](https://www.docker.com/products/docker-desktop/)
-   [Dapr CLI](https://docs.dapr.io/getting-started/install-dapr-cli/)
-   [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)

*(Detailed setup and run instructions will be added here as development progresses.)*