# Phase 1 Spec: Core Chat Assistant UI

**Component:** Frontend (`/frontend`)  
**Source Plan:** [`docs/plans/Phased_Plan.md`](/docs/plans/Phased_Plan.md)

## 1. Goal

To transform the basic status dashboard into a fully functional, real-time chat application. This involves building the core UI components for conversation, managing session state, and handling streaming data from the backend.

## 2. Feature Breakdown & Acceptance Criteria

### 2.1. Conversational UI

-   **Goal:** An intuitive and clean interface for users to interact with the AI.
-   **Acceptance Criteria:**
    -   [ ] The main view is a chat interface.
    -   [ ] A scrollable message list displays the conversation history. User messages are visually distinct from AI messages (e.g., alignment, color).
    -   [ ] A text input area at the bottom allows users to type messages.
    -   [ ] A "Send" button submits the message. Pressing "Enter" in the text area also submits the message.
    -   [ ] The "Send" button is disabled while the AI is generating a response to prevent duplicate submissions.

### 2.2. Streaming Response Handling

-   **Goal:** To provide a responsive user experience by displaying the AI's response as it is generated.
-   **Acceptance Criteria:**
    -   [ ] The frontend client correctly initiates and reads a streaming `fetch` request from the `/v1/chat` endpoint.
    -   [ ] As text chunks are received from the stream, they are appended to the AI's message bubble in the UI, creating a "live typing" effect.
    -   [ ] The UI remains interactive and does not freeze while the response is streaming.

### 2.3. Session and State Management

-   **Goal:** To manage the state of the conversation, including history, sessions, and error conditions.
-   **Acceptance Criteria:**
    -   [ ] **Session Handling:**
        -   [ ] On first load, a unique `conversationId` is generated client-side.
        -   [ ] A "New Chat" button is implemented. Clicking it clears the current message history and generates a new `conversationId`.
        -   [ ] (Optional for this phase, but good to consider) A session switcher UI is added to the sidebar, listing recent conversations and allowing the user to switch between them.
    -   [ ] **Error Handling:**
        -   [ ] If an API call fails, a user-friendly error message is displayed in the chat interface (e.g., "An error occurred. Please try again.").
        -   [ ] A "Regenerate" button appears next to the last AI response, allowing the user to request a new answer for their last prompt.
        -   [ ] A "Resend" button appears next to a user message if it failed to send.

### 2.4. Basic Observability

-   **Goal:** To enable end-to-end tracing of user interactions.
-   **Acceptance Criteria:**
    -   [ ] A unique correlation ID (e.g., a UUID) is generated for each user request initiated from the frontend.
    -   [ ] This correlation ID is passed as an HTTP header in the request to the backend API.

## 3. Definition of Done (DoD)

-   [ ] A user can open the web application and start a conversation with the AI.
-   [ ] The AI's response streams into the UI in real-time.
-   [ ] The entire conversation is persisted (by the backend) and can be reloaded by the client if the page is refreshed (stretch goal for this phase, but required by DoD).
-   [ ] The user can start a new, separate conversation at any time.
-   [ ] The UI provides clear feedback for loading and error states.