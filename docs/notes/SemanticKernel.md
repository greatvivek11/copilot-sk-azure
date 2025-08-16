# Technical Primer: Semantic Kernel

This document provides a detailed, project-agnostic overview of Microsoft's Semantic Kernel (SK), focusing on its core concepts and practical application for building sophisticated AI agents.

---

## 1. The Kernel: The AI Orchestration Engine

The `Kernel` is the heart of Semantic Kernel. It's the engine that orchestrates AI models, plugins, and memory to execute complex tasks.

-   **Core Functionality**: The kernel is responsible for managing services (like AI models), loading plugins, and orchestrating the execution flow. It doesn't perform any actions itself but delegates to the appropriate components.
-   **Dependency Injection**: In a typical ASP.NET Core application, the `Kernel` is registered as a service and injected wherever AI capabilities are needed, promoting a clean and testable architecture.

---

## 2. Plugins and Functions: The Building Blocks of AI Skills

Plugins are collections of functions that define the AI's capabilities. Functions are the smallest unit of work and can be either "semantic" (natural language prompts) or "native" (C# code).

### 2.1. Semantic Functions (Prompts)

Prompts are the primary way to interact with Large Language Models (LLMs). Semantic Kernel enhances this with a powerful templating system.

*   **Prompt Engineering**: This is the art of crafting effective prompts. Good prompts are clear, specific, and provide sufficient context for the model to generate the desired output.
*   **Prompt Templates**: SK uses templates to create reusable and dynamic prompts.
    *   **Basic Syntax**: Uses `{{$input}}` for simple variable substitution.
    *   **Advanced Templating**: Supports **Handlebars** and **Liquid** syntax for more complex logic like loops and conditionals within the prompt itself. This is powerful for dynamically constructing prompts based on complex inputs.
*   **YAML Schema**: Prompts can be defined in `.skprompt.yml` files, which separate the prompt logic from the C# code. This allows prompt engineers and developers to work independently. The schema allows for defining:
    *   `template_format`: (e.g., `handlebars`, `liquid`)
    *   `input_variables`: Defines the expected inputs for the prompt.
    *   `execution_settings`: Allows specifying different model parameters (like `temperature`) for different AI models.

### 2.2. Native Functions (C# Code)

Native functions are standard C# methods that allow the AI to interact with external systems.

*   **`[KernelFunction]` Attribute**: Marks a C# method as a function that can be called by the kernel.
*   **`[Description]` Attribute**: Crucial for planners. The description tells the AI *what the function does* in natural language, enabling the planner to decide when to use it.
*   **Use Cases**:
    *   Accessing a database to retrieve customer data.
    *   Calling an external REST API (e.g., a weather service).
    *   Interacting with the local file system.

---

## 3. Planners: Automated Task Orchestration

Planners enable agent-like behaviors by creating a sequence of steps to achieve a goal. Modern LLMs with **Function Calling** capabilities have streamlined this process.

*   **Automatic Function Calling**: This is the modern approach to planning in Semantic Kernel. Instead of generating a textual plan, the LLM directly outputs a request to call one or more specific functions from the available plugins.
    *   The developer enables this by setting `FunctionChoiceBehavior.Auto()` in the prompt execution settings.
    *   The kernel handles the loop:
        1.  Send the user's request and available function definitions to the LLM.
        2.  LLM responds with either a message or a request to call a function.
        3.  If it's a function call, the kernel executes the native code.
        4.  The result of the function is sent back to the LLM.
        5.  The LLM uses the result to generate the final response.

This automates the complex orchestration logic, allowing developers to focus on defining the right tools (plugins) for the agent.

---

## 4. Connectors and Memory: The Bridge to External Knowledge

Connectors link the kernel to external services, including AI models and data stores.

### 4.1. AI Service Connectors

These connectors bridge the kernel to various AI models.
*   **Chat Completion**: `IChatCompletionService` for interacting with chat-based models like Gemini or GPT-4.
*   **Text Embedding**: `ITextEmbeddingGenerationService` for converting text into numerical vectors, which is the foundation of semantic search.

### 4.2. Memory and Retrieval-Augmented Generation (RAG)

Memory is what gives an AI long-term knowledge and the ability to reason over specific data. This is achieved through Retrieval-Augmented Generation (RAG).

*   **The RAG Process**:
    1.  **Ingestion**: Documents (e.g., PDFs, Word docs) are loaded and broken down into smaller, meaningful text chunks.
    2.  **Embedding**: Each chunk is passed to an embedding model, which converts it into a vector (a numerical representation of its semantic meaning).
    3.  **Storage**: These vectors are stored in a specialized **Vector Database** (e.g., Qdrant, Azure AI Search).
    4.  **Retrieval**: When a user asks a question, their query is also embedded. The vector database performs a similarity search to find the text chunks with vectors most similar to the query's vector.
    5.  **Augmentation**: These relevant chunks are retrieved and injected into the prompt that is sent to the LLM, providing it with the necessary context to answer the question accurately.

*   **Vector Store Connectors**: Semantic Kernel provides abstractions (`IVectorStore`) and connectors for various vector databases, simplifying the implementation of RAG. This allows you to easily switch between different storage backends (e.g., from an in-memory store for local testing to a production-grade database in the cloud).