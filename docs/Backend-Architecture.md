# 🏛️ Backend Architecture

This document outlines the architectural decisions and patterns for the backend service of the AI-Powered Knowledge Hub. The goal is to create a modern, maintainable, and scalable application that is easy to develop and test, effectively showcasing advanced C# and .NET skills.

---

## 🎯 Core Principles

1.  **Organize by Feature, Not by Technical Layer**: We will group code by its business capability (feature) rather than its technical role (e.g., "Services", "Repositories").
2.  **Pragmatism over Dogma**: The architecture should serve the project's needs without adding unnecessary complexity. We will avoid patterns that add more ceremony than value for this specific scope.
3.  **Single Project Simplicity**: We will maintain all backend code within a single ASP.NET Core project to maximize simplicity and development speed.

---

## 🏗️ Architectural Pattern: Vertical Slice Architecture (VSA)

We will be using **Vertical Slice Architecture (VSA)** as our primary organizational pattern.

### What is VSA?

VSA is a way of structuring code where all the components required for a single feature or use case are located together. This includes the API endpoint, business logic, data access, and any request/response models.

### VSA vs. Clean/Onion Architecture

VSA is not a replacement for the principles of Clean Architecture (like separation of concerns), but rather a different approach to applying them.

-   **Clean/Onion Architecture (Horizontal Slicing)**: Organizes code into technical layers (e.g., `Project.Domain`, `Project.Application`, `Project.Infrastructure`). This forces you to work across multiple projects for a single feature.
-   **Vertical Slice Architecture**: Organizes code by feature. This promotes high cohesion within a feature and low coupling between features.

For this project, **we will use VSA within a single project**, which gives us the conceptual benefits of separation without the physical complexity of multiple class libraries.

### Why VSA for This Project?

-   **High Cohesion**: All code related to a feature lives in one place, making it easier to find, understand, and modify.
-   **Low Coupling**: Slices are self-contained and have minimal dependencies on each other. Changing one feature is unlikely to break another.
-   **Improved Developer Experience**: Reduces context-switching and the need to navigate a complex project structure.

---

## 🔍 Architectural Refinements

While VSA provides the core organizational structure, we will introduce a few key abstractions to improve maintainability, testability, and separation of concerns.

1.  **Shared Infrastructure Layer**: We will create a dedicated `Infrastructure` folder to house the implementation details of external services. This keeps our Feature slices clean and focused on business logic.
2.  **AI Service Abstraction**: We will wrap the Semantic Kernel and its model provider setup behind our own interface (e.g., `IAiService`). This decouples the application from the specific AI implementation, making it easy to swap between the Hugging Face Router and Azure OpenAI in the future.
3.  **Deferred Async Messaging**: For long-running tasks like document ingestion, we acknowledge that an asynchronous, event-driven approach using Dapr Pub/Sub would be ideal. However, to manage complexity, we will start with synchronous implementations and defer the introduction of async messaging until a later phase.

---

## 📦 Evolved Project Structure

This structure incorporates the refined architectural decisions.

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
│   │   └── SemanticKernelService.cs  // Implementation using SK
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
└── Program.cs                        // Composition Root
```

### Folder & Layer Responsibilities

1.  **`Domain/`**: Unchanged. Contains the core POCO entities.

2.  **`Features/`**: Remains the heart of the application.
    -   **Key Change**: Feature handlers will **no longer interact directly with low-level APIs** like `DbContext` or Azure SDKs. Instead, they will declare dependencies on the clean abstractions defined in the `Infrastructure` layer (e.g., `IAiService`, `IAzureBlobStorage`) and receive them via dependency injection.

3.  **`Infrastructure/`**: The new home for all external-facing concerns.
    -   This folder contains the concrete implementations of the interfaces it defines.
    -   **`Infrastructure/Ai/`**: Contains the `SemanticKernelService` which implements `IAiService`. All the complexity of setting up the kernel, connectors, and prompts is encapsulated here.
    -   **`Infrastructure/Persistence/`**: Contains the EF Core `DbContext` and handles all database-related concerns.
        -   **EF Core & Code-First Migrations**: We will use a **code-first** approach. The `DbContext` and its entity configurations define the desired database schema.
        -   **Migrations**: We will use `dotnet ef migrations add` to generate new migration scripts when the model changes. These scripts are checked into source control.
        -   **Applying Migrations**: Migrations are applied automatically as part of the CI/CD pipeline, just before the new application version is deployed. This ensures the database schema is always in sync with the application's expectations.
    -   **`Infrastructure/Storage/`**: Contains the implementation for interacting with Azure Blob Storage.
    -   **Dependency Inversion**: This layer follows the Dependency Inversion Principle. It defines interfaces and then provides their implementations, which are then injected into the `Features` layer.

4.  **`Shared/`**: Unchanged. For genuinely cross-cutting concerns like MediatR behaviors.

5.  **`Program.cs`**: The Composition Root.
    -   **Responsibility**: Its primary role is to register all the interface-to-implementation mappings from the `Infrastructure` layer (e.g., `builder.Services.AddScoped<IAiService, SemanticKernelService>();`). This is where the application is "composed."