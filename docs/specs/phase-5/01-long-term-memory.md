# Phase 5 Spec: Long-Term Memory

**Component:** Backend (`/backend`)  
**Source Plan:** [`docs/plans/Phased_Plan.md`](/docs/plans/Phased_Plan.md)

## 1. Goal

To implement a long-term memory system for the AI assistant. This involves creating a process to summarize past conversations and a mechanism to retrieve those summaries to provide context for new conversations, making the assistant more personalized and context-aware.

## 2. Feature Breakdown & Acceptance Criteria

### 2.1. Conversation Summarization Job

-   **Goal:** An automated, asynchronous process to distill and store the essence of a conversation.
-   **Acceptance Criteria:**
    -   [ ] A scheduled, background job is created. This can be implemented as a time-triggered **Azure Container Apps Job**.
    -   [ ] The job runs periodically (e.g., every hour) and queries the Azure SQL database for conversations that have recently ended or been inactive for a certain period.
    -   [ ] For each completed conversation, the job retrieves the full message history.
    -   [ ] A **Semantic Function** is invoked, with a prompt specifically designed to summarize the conversation into a concise, factual summary. The prompt should instruct the model to focus on key entities, user preferences, and resolutions.
    -   [ ] The generated summary text is then converted into a vector embedding.
    -   [ ] The summary text and its corresponding vector are stored in a dedicated **Azure Cosmos DB** collection for long-term memory, linked to the user's ID.

### 2.2. Memory-Augmented Context

-   **Goal:** To use the stored memories to inform and personalize new conversations.
-   **Acceptance Criteria:**
    -   [ ] When a user starts a new chat session, the backend identifies the user.
    -   [ ] An initial query (e.g., the user's first message, or even just the user's ID) is embedded.
    -   [ ] A similarity search is performed against the Cosmos DB memory collection to find the `top-N` most relevant past conversation summaries for that user.
    -   [ ] These retrieved summaries are then **injected into the system prompt** of the new conversation.
    -   [ ] The system prompt is engineered to instruct the main chat LLM on how to use this memory, e.g., "You are a helpful assistant. Here is a summary of your past conversations with this user. Use it to personalize your responses: [retrieved summaries]".

### 2.3. Frontend Indication (Optional but Recommended)

-   **Goal:** To subtly inform the user that the AI is using memory.
-   **Acceptance Criteria:**
    -   [ ] When a new chat starts and memory is successfully loaded, the UI could display a small, unobtrusive message like "Welcome back! I remember our last conversation about..." or simply a "Context loaded" indicator.

## 3. Definition of Done (DoD)

-   [ ] After a conversation is completed, a background job successfully generates a summary and stores it as a vector in Cosmos DB.
-   [ ] When a user starts a new conversation, the AI's first response demonstrates awareness of a key point from a previous conversation.
-   [ ] The system prompt for a new chat session correctly includes the retrieved memory summaries.
-   [ ] The process is efficient and does not introduce significant latency to the start of a new chat session.