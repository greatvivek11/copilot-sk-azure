# Phase 1 Spec: Core Chat Assistant Backend

**Component:** Backend (`/backend`)  
**Source Plan:** [`docs/plans/Phased_Plan.md`](/docs/plans/Phased_Plan.md)

## 1. Goal

To implement the core backend functionality for a real-time, conversational AI chat. This includes creating a streaming API endpoint, persisting the conversation history to a database, and introducing a lightweight CQRS pattern using a source-generated Mediator for clean, maintainable code.

## 2. Feature Breakdown & Acceptance Criteria

### 2.1. Domain Model & Persistence Schema

-   **Goal:** To establish the database schema for storing all conversation data.
-   **Acceptance Criteria:**
    -   [ ] New EF Core entities are created for `User`, `Conversation`, and `Message`.
    -   [ ] Relationships are defined: A `User` can have multiple `Conversations`, and a `Conversation` has a list of `Messages`.
    -   [ ] EF Core migrations are generated for the new tables and relationships. These migrations are checked into source control.
    -   [ ] The CI/CD pipeline is updated to automatically apply EF Core migrations to the Azure SQL database during deployment.

### 2.2. Conversation Persistence Logic

-   **Goal:** To reliably save every user message and AI response to the database to maintain a complete conversation history.
-   **Acceptance Criteria:**
    -   [ ] The `Azure SQL Database` schema is updated (using EF Core migrations) to include tables for:
        -   `ChatSessions`: To store metadata about each conversation.
        -   `ChatMessages`: To store each individual user message and AI response, linked to a session.
    -   [ ] When a new chat session starts, a new record is created in the `ChatSessions` table.
    -   [ ] Every user message and its corresponding AI response is saved as a record in the `ChatMessages` table.
    -   [ ] The backend can retrieve the message history for a given chat session ID to provide context to the AI model for subsequent turns in the conversation.

### 2.3. Streaming Chat Endpoint

-   **Goal:** A robust API endpoint that can handle user messages and stream back AI responses in real-time.
-   **Acceptance Criteria:**
    -   [ ] A new feature slice directory, `/Features/Chat`, is created.
    -   [ ] A `POST` endpoint is defined at `/v1/chat`.
    -   [ ] The endpoint accepts a user's message and a `conversationId`.
    -   [ ] The endpoint uses Semantic Kernel to invoke the chat model, providing the new message and the historical context retrieved from the database for that `conversationId`.
    -   [ ] The AI's response is streamed back to the client using `IAsyncEnumerable<string>`.

### 2.4. Lightweight CQRS with Mediator

-   **Goal:** To structure the application logic for better separation of concerns and testability, using a modern, high-performance Mediator pattern.
-   **Acceptance Criteria:**
    -   [ ] A source-generated Mediator library is added to the project instead of the reflection-based MediatR library.
    -   [ ] The logic for handling the chat request is encapsulated in a `Command` or `Request` object (e.g., `StreamChatRequest`).
    -   [ ] A corresponding `Handler` (e.g., `StreamChatRequestHandler`) contains the core business logic: retrieving history, calling the AI, and triggering the persistence of new messages.
    -   [ ] The Minimal API endpoint for `/v1/chat` becomes a thin wrapper that simply creates the request object and sends it to the mediator.

### 2.5. Basic Observability

-   **Goal:** To ensure that the chat flow can be traced and debugged.
-   **Acceptance Criteria:**
    -   [ ] A correlation ID is expected from the frontend (or generated if not present) and is logged with all related log entries for a single request.
    -   [ ] Structured logs are emitted at the start and end of the chat handler, including PII-safe metadata like `conversationId` and response time.

## 3. Definition of Done (DoD)

-   [ ] A user can have multiple, concurrent chat sessions, and the history for each is correctly maintained and isolated.
-   [ ] When a client reconnects with a valid `conversationId`, the message history is correctly loaded and displayed.
-   [ ] The streaming response from the AI is smooth and responsive on the frontend.
-   [ ] The code structure clearly separates the API endpoint from the business logic using the source-generated Mediator pattern.