# 🏛️ Backend Architecture

This document outlines the architectural decisions and patterns for the backend service of the AI-Powered Knowledge Hub. The goal is to create a modern, maintainable, and scalable application that is easy to develop and test, effectively showcasing advanced C# and .NET skills.

## 🎯 Core Principles

1. **Organize by Feature, Not by Technical Layer**: We will group code by its business capability (feature) rather than its technical role (e.g., "Services", "Repositories").
2. **Pragmatism over Dogma**: The architecture should serve the project's needs without adding unnecessary complexity. We will avoid patterns that add more ceremony than value for this specific scope.
3. **Single Project Simplicity with Aspire Orchestration**: We will maintain all backend code within a single ASP.NET Core project while using **.NET Aspire** to compose, configure, and run distributed services across ACA.

## 🏗️ Architectural Pattern: Vertical Slice Architecture (VSA)

We will be using **Vertical Slice Architecture (VSA)** as our primary organizational pattern.

### What is VSA?

VSA is a way of structuring code where all the components required for a single feature or use case are located together. This includes the API endpoint, business logic, data access, and any request/response models.

### VSA vs. Clean/Onion Architecture

VSA is not a replacement for the principles of Clean Architecture (like separation of concerns), but rather a different approach to applying them.

* **Clean/Onion Architecture (Horizontal Slicing)**: Organizes code into technical layers (e.g., `Project.Domain`, `Project.Application`, `Project.Infrastructure`). This forces you to work across multiple projects for a single feature.
* **Vertical Slice Architecture**: Organizes code by feature. This promotes high cohesion within a feature and low coupling between features.

For this project, **we will use VSA within a single project**, but managed and deployed with **Aspire AppHost** for configuration and local orchestration. This gives us the conceptual benefits of separation while enabling modern cloud-native deployment practices.

### Why VSA for This Project?

* **High Cohesion**: All code related to a feature lives in one place, making it easier to find, understand, and modify.
* **Low Coupling**: Slices are self-contained and have minimal dependencies on each other. Changing one feature is unlikely to break another.
* **Improved Developer Experience**: Reduces context-switching and the need to navigate a complex project structure.
* **Aspire Alignment**: Keeps service orchestration declarative while maintaining backend code simple and feature-focused.

## 🔍 Architectural Refinements

While VSA provides the core organizational structure, we will introduce a few key abstractions to improve maintainability, testability, and separation of concerns.

1. **Shared Infrastructure Layer**: Dedicated `Infrastructure` folder houses implementation details of external services. Aspire config (e.g., service bindings, health checks, env vars) augments but does not replace this layer.
2. **AI Service Abstraction**: Wrap Semantic Kernel and model providers behind `IAiService`. The implementation (`SemanticKernelService`) integrates with Hugging Face Router via OpenAI-compatible APIs. This decouples features from provider choice.
3. **Dapr Integration**: Interservice communication and pub/sub are handled with **Dapr sidecars**, invoked internally via app-id addressing.
4. **Async Messaging Deferred**: For long-running tasks (like ingestion/embedding), we plan eventual use of Dapr Pub/Sub. Initially, synchronous workflows will keep complexity manageable.
5. **Event-Driven Extension Path**: Aspire + Dapr make it easy to evolve into a microservices/event-driven setup in later phases without major rewrites.

## 📦 Evolved Project Structure

```
/src/AIHub.Backend/
|
├── Domain/
│   ├── Document.cs
│   └── ChatSession.cs
│
├── Features/
│   ├── IngestDocument/
│   │   ├── IngestDocument.cs
│   │   ├── IngestDocumentHandler.cs  // Depends on IAzureBlobStorage, IAiService
│   │   └── Endpoint.cs
│   └── ...
│
├── Infrastructure/
│   ├── Ai/
│   │   ├── IAiService.cs             // Interface for AI orchestration
│   │   └── SemanticKernelService.cs  // Implementation using SK + HF Router
│   ├── Persistence/
│   │   ├── AIHubDbContext.cs
│   │   └── Migrations/
│   └── Storage/
│       ├── IAzureBlobStorage.cs      // Interface for blob operations
│       └── AzureBlobStorage.cs       // Implementation using Azure SDK
│
├── Shared/
│   └── ...
│
├── Aspire/
│   └── AppHost.cs                    // Aspire orchestration and configuration
│
└── Program.cs                        // Minimal APIs + Composition Root
```

### Folder & Layer Responsibilities

1. **`Domain/`**: Contains the core POCO entities.

2. **`Features/`**: Heart of the application.

   * Handlers depend only on abstractions (e.g., `IAiService`, `IAzureBlobStorage`).
   * Aspire manages runtime bindings and secrets injection.

3. **`Infrastructure/`**:

   * **AI**: `SemanticKernelService` wraps SK with Hugging Face Router/OpenAI endpoints.
   * **Persistence**: EF Core `DbContext` (code-first migrations, auto-applied in CI/CD).
   * **Storage**: Azure Blob interactions.
   * **Dapr**: Dapr client abstractions for service invocation/pub-sub.

4. **`Shared/`**: Cross-cutting concerns like behaviors, middleware.

5. **`Aspire/`**: Configures services, environment bindings, secrets, ACA integration.

6. **`Program.cs`**: Minimal APIs serve as the entry point. Registers services, slices, Dapr sidecar integration, telemetry (OTel), health checks, etc.

## 🚀 Key Improvements with Aspire

* **Configuration-as-Code**: Aspire AppHost orchestrates ACA, CosmosDB, Blob, and SQL dependencies.
* **Local Dev Parity**: Aspire runs all services locally with Dapr sidecars for testing.
* **Cloud-Native Ready**: Easy to extend into event-driven patterns without restructuring.
* **Minimal APIs First**: Fast to implement, still production-ready.
* **Telemetry Built-In**: OTel integration with Aspire surfaces traces across FE/BE services.

## ✅ Success Criteria

* Vertical slices implemented in single project.
* Aspire AppHost orchestrates ACA containers and Dapr sidecars.
* Minimal APIs expose feature endpoints with streaming support.
* Feature handlers depend only on abstractions, not concrete infra.
* Ready for extension into async/event-driven model in later phases.
