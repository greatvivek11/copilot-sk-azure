# Phase 2 Spec: Document Ingestion & RAG Pipeline

**Component:** Backend (`/backend`) & Platform  
**Source Plan:** [`docs/plans/Phased_Plan.md`](/docs/plans/Phased_Plan.md)

## 1. Goal

To build the complete backend pipeline that enables the AI to answer questions based on user-uploaded documents. This involves creating a secure file upload mechanism, a robust ingestion worker, and enhancing the chat endpoint to perform Retrieval-Augmented Generation (RAG).

## 2. Feature Breakdown & Acceptance Criteria

### 2.1. Secure Upload Endpoint

-   **Goal:** A secure mechanism for the frontend to upload files directly to Blob Storage without exposing storage keys.
-   **Acceptance Criteria:**
    -   [ ] A `POST` endpoint is created at `/v1/uploads/signed-url`.
    -   [ ] The endpoint accepts a `fileName` in the request body.
    -   [ ] It generates a short-lived (e.g., 5 minutes) **Shared Access Signature (SAS) URL** for `Azure Blob Storage` with `write` permission.
    -   [ ] The endpoint returns the SAS URL to the client.
    -   [ ] After the frontend confirms the upload, a corresponding `Document` metadata record is created in the Azure SQL database (e.g., storing `documentId`, `fileName`, `status: 'Uploaded'`).

### 2.2. Ingestion Worker (ACA Job)

-   **Goal:** An asynchronous, reliable process for extracting, chunking, and embedding document content.
-   **Acceptance Criteria:**
    -   [ ] A new, separate containerized application is created to act as the ingestion worker. This will be deployed as an **Azure Container Apps Job**.
    -   [ ] A `POST` endpoint is created at `/v1/ingest`. Calling this endpoint triggers the ingestion job for a specific document.
    -   [ ] **Text Extraction:** The worker can extract clean text from `.txt`, `.pdf`, and `.docx` files using open-source .NET libraries.
    -   [ ] **Chunking:** The extracted text is divided into smaller, token-aware chunks (e.g., targeting ~200-300 tokens per chunk with some overlap).
    -   [ ] **Embedding:** Each text chunk is sent to the Hugging Face embedding model via Semantic Kernel.
    -   [ ] **Vector Upsert:** The text chunk, its vector embedding, and source metadata (e.g., `documentId`, `pageNumber`, `chunkId`) are saved as a new document in the **Azure Cosmos DB** vector collection.
    -   [ ] The `Document`'s status in the SQL database is updated to `'Processed'` upon successful completion.

### 2.3. RAG-Enabled Chat Endpoint

-   **Goal:** To enhance the existing chat endpoint to use the vector store for grounded, cited answers.
-   **Acceptance Criteria:**
    -   [ ] The `/v1/chat` endpoint is modified to accept a `mode` or similar parameter to distinguish between general chat and document-based Q&A.
    -   [ ] In "Ask about docs" mode, the following RAG process is executed:
        1.  The user's query is converted into a vector embedding.
        2.  A similarity search is performed on the Cosmos DB vector store to retrieve the top-k most relevant text chunks.
        3.  The retrieved chunks are injected as context into the prompt sent to the main chat LLM.
        4.  The prompt is engineered to instruct the model to answer **only** based on the provided context and to cite its sources.
    -   [ ] The final JSON response from the API includes not just the answer, but also an array of `citations`, each containing metadata about the source chunk (e.g., `documentName`, `pageNumber`).

## 3. Definition of Done (DoD)

-   [ ] A user can upload a document, and the ingestion worker successfully processes it and populates the Cosmos DB vector store.
-   [ ] When asking a question about the document's content, the user receives a relevant answer.
-   [ ] The answer is accompanied by 2-5 accurate citations pointing to the source document.
-   [ ] The entire process, from upload to RAG-based answer, is functional in the deployed Azure environment.