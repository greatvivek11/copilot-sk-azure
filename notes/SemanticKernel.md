# 🧠 Understanding Semantic Kernel: Core Concepts

This document provides a foundational understanding of Microsoft's Semantic Kernel (SK), a lightweight, open-source SDK that helps you orchestrate AI models like Google Gemini. Think of it as a "brain" for your application, connecting your code to AI services.

---

## 1. The Kernel: The "Brain" 🧠

The `Kernel` is the central object in Semantic Kernel. It's the orchestrator that processes requests by chaining together different components.

-   **What it does**: Manages plugins, services, and memory to fulfill a user's request.
-   **Analogy**: It's like a CPU for your AI logic. You load it with "software" (plugins) and give it access to "memory," and it runs the program.

**In our project**, the `Kernel` will be configured in `Program.cs` and injected via dependency injection wherever we need to perform an AI task.

```csharp
// Register the SK Kernel as a transient service
builder.Services.AddTransient<Kernel>();

// Later, in an API endpoint...
app.MapPost("/api/chat", async (ChatRequest req, Kernel kernel) => {
    // ... use the kernel to run a function
});
```

---

## 2. Plugins: The "Skills" 🛠️

A **Plugin** is a container for a collection of **Functions**. They are the building blocks of your AI's capabilities and can be composed to create complex workflows.

There are two main types of functions:

### a. Semantic Functions (Prompts)

-   **What they are**: Natural language prompts defined in simple text files (`skprompt.txt`). They are essentially instructions for the LLM.
-   **Structure**: A prompt file contains the prompt template with placeholders (`{{$input}}`) and a `config.json` file that defines its parameters (e.g., temperature, top_p, input variables).
-   **Use Case**: Summarizing text, extracting entities, answering questions, changing writing tone.

**Example `skprompt.txt` for summarization:**

```
Summarize the following text in one sentence:

{{$input}}
```

### b. Native Functions (C# Code)

-   **What they are**: Regular C# methods decorated with the `[KernelFunction]` attribute.
-   **What they do**: Allow your AI to interact with the "real world" by calling your code.
-   **Use Case**: Accessing a database, calling a web API, reading a file, performing complex calculations—anything you can write in C#.

**Example Native Function:**

```csharp
public class DocumentPlugin
{
    [KernelFunction]
    [Description("Reads a file from the local file system.")]
    public async Task<string> ReadFile(
        [Description("The full path to the file.")] string filePath
    )
    {
        return await File.ReadAllTextAsync(filePath);
    }
}
```

---

## 3. Planners: The "Orchestrators" 🗺️

A **Planner** is a special function that uses the AI model to create a **Plan**—a dynamic sequence of functions to achieve a complex goal.

-   **How it works**: You give the planner a goal (e.g., "Summarize the document at 'C:/report.pdf' and then email the summary to my manager."). The planner looks at all available plugins and figures out which functions to call in what order to achieve the goal.
-   **The Plan**: The output is an executable plan that the kernel can run. For the example above, it might look like:
    1.  `DocumentPlugin.ReadFile("C:/report.pdf")`
    2.  `SummarizePlugin.Summarize(input_from_step_1)`
    3.  `EmailPlugin.SendEmail(to="manager@...", body=input_from_step_2)`

**Planners enable agent-like behaviors**, where the AI can reason and decide how to solve a problem on its own.

---

## 4. Connectors: The "Bridges" 🌉

**Connectors** are the bridges that link the kernel to external services.

### a. AI Service Connectors

-   **What they do**: Connect to AI models for different modalities.
-   **Examples**:
    -   `IChatCompletionService`: For chat-based models (like Gemini).
    -   `ITextEmbeddingGenerationService`: For generating vector embeddings for text.
-   **In our project**, we use `AddGoogleAIGeminiChatCompletion` to register the Gemini connector.

### b. Memory & Vector Store Connectors

-   **What they do**: Connect to data sources where your AI can store and retrieve information. This is the foundation of **Retrieval-Augmented Generation (RAG)**.
-   **How it works**:
    1.  **Ingestion**: Text is broken into chunks.
    2.  **Embedding**: Each chunk is converted into a vector (a list of numbers) using an embedding model.
    3.  **Storage**: These vectors are stored in a **Vector Database**.
    4.  **Retrieval**: When a user asks a question, their query is also embedded, and the database finds the most similar (relevant) text chunks.
-   **Connectors in our project**:
    -   `InMemoryVectorStore`: For fast, local prototyping.
    -   `Qdrant`: For a persistent, production-ready vector database.

---

## Summary: How It All Fits Together

1.  A user makes a **request**.
2.  The **Kernel** receives the request.
3.  The Kernel might use a **Planner** to create a multi-step plan.
4.  The plan executes a sequence of **Functions** from various **Plugins**:
    -   A **Native Function** might be called to fetch data from an API.
    -   A **Semantic Function** might be called to summarize that data.
5.  The Kernel uses **Connectors** to talk to **AI Services** (like Gemini) and **Memory** (like Qdrant).
6.  The final result is returned to the user.

This modular architecture allows you to build sophisticated, scalable, and maintainable AI applications.