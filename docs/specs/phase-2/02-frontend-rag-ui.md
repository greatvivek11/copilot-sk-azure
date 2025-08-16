 # Phase 2 Spec: Document Upload & RAG Chat UI

**Component:** Frontend (`/frontend`)  
**Source Plan:** [`docs/plans/Phased_Plan.md`](/docs/plans/Phased_Plan.md)

## 1. Goal

To build the user interface that allows users to upload their documents and interact with the new RAG-powered chat mode, including the clear presentation of source citations.

## 2. Feature Breakdown & Acceptance Criteria

### 2.1. Document Upload Page

-   **Goal:** A dedicated, user-friendly interface for managing and uploading documents.
-   **Acceptance Criteria:**
    -   [ ] A new page/route (e.g., `/uploads`) is created.
    -   [ ] The page features a prominent file upload area (e.g., a drag-and-drop zone or a file selection button).
    -   [ ] The file input is restricted to the supported types: `.pdf`, `.docx`, `.txt`.
    -   [ ] **Upload Process:**
        1.  User selects a file.
        2.  The frontend calls the backend's `/v1/uploads/signed-url` endpoint to get the secure SAS URL.
        3.  The frontend uses the `fetch` API with `PUT` to upload the file directly to the returned SAS URL.
        4.  An upload progress bar or spinner is displayed for the file being uploaded.
        5.  Once the upload is complete, the frontend calls the backend's `/v1/ingest` endpoint to trigger the processing pipeline.
    -   [ ] The page displays a list of previously uploaded documents and their current processing status (e.g., `Uploading`, `Processing`, `Ready`). This status is updated by polling the backend or through a real-time notification system (polling is sufficient for this phase).

### 2.2. RAG Chat Mode UI

-   **Goal:** To seamlessly integrate the document-aware chat mode into the existing chat interface.
-   **Acceptance Criteria:**
    -   [ ] A `Switch` or `Toggle` component is added to the chat UI, allowing the user to switch between "General Chat" and "Ask About Docs" modes.
    -   [ ] When "Ask About Docs" mode is selected, the chat input's placeholder text changes to something descriptive, like "Ask a question about your uploaded documents...".
    -   [ ] The `mode: "rag"` parameter is included in all chat requests sent to the backend while this mode is active.

### 2.3. Citation Display

-   **Goal:** To clearly and transparently show the user where the AI's information is coming from.
-   **Acceptance Criteria:**
    -   [ ] When the AI's response is rendered, the frontend checks for the `citations` array in the API response.
    -   [ ] If citations are present, they are displayed clearly beneath the AI's message.
    -   [ ] Each citation is rendered as a distinct visual element (e.g., a "chip" or "tag") showing the source document name and page number (e.g., `[report.pdf, page 2]`).
    -   [ ] (Stretch Goal) Clicking on a citation could trigger a modal or a side panel showing the full text of the source chunk that was used.

## 3. Definition of Done (DoD)

-   [ ] A user can navigate to the uploads page, select a valid document, and see it successfully upload and get processed.
-   [ ] In the chat UI, the user can switch to "Ask About Docs" mode.
-   [ ] When a question is asked in this mode, the AI's answer is displayed along with accurate, clearly formatted source citations.
-   [ ] The UI gracefully handles all loading states (uploading, processing, waiting for AI response) and error conditions.