# Phase 3 Spec: Vision & OCR

**Component:** Backend (`/backend`) & Frontend (`/frontend`)  
**Source Plan:** [`docs/plans/Phased_Plan.md`](/docs/plans/Phased_Plan.md)

## 1. Goal

To extend the ingestion pipeline and RAG capabilities to include visual data. This involves analyzing the content of images and performing Optical Character Recognition (OCR) on documents that are image-based (e.g., scanned PDFs), making their content searchable.

## 2. Feature Breakdown & Acceptance Criteria

### 2.1. Image Content Analysis

-   **Goal:** To understand the content of uploaded images so they can be searched semantically.
-   **Acceptance Criteria:**
    -   [ ] The frontend upload component is updated to accept common image formats (e.g., `.png`, `.jpg`, `.jpeg`).
    -   [ ] The backend ingestion worker (ACA Job) identifies image content types.
    -   [ ] For images, the worker calls a multimodal vision model via the Hugging Face router.
    -   [ ] The model generates a rich, textual description of the image's content (e.g., "A bar chart showing a 25% increase in Q3 sales for the Alpha project").
    -   [ ] This generated text description is then sent through the existing Phase 2 pipeline: it is chunked, embedded, and upserted into the Cosmos DB vector store.
    -   [ ] The metadata stored with the chunk includes a reference to the original image URI in Blob Storage.

### 2.2. OCR for Scanned Documents

-   **Goal:** To extract and index text from documents that do not have selectable text layers.
-   **Acceptance Criteria:**
    -   [ ] The ingestion worker can differentiate between a text-based PDF and an image-based (scanned) PDF.
    -   [ ] For scanned PDFs, the worker sends each page to an OCR service/model.
    -   [ ] The chosen OCR solution can be a containerized open-source tool like **Tesseract** or **PaddleOCR**, deployed as another internal service in the ACA environment, or a call to a Hugging Face model endpoint.
    -   [ ] The raw text extracted from each page is collected.
    -   [ ] The aggregated text for the entire document is then passed to the standard chunking and embedding pipeline from Phase 2.
    -   [ ] The system correctly handles multi-page scanned documents.

### 2.3. Frontend Enhancements

-   **Goal:** To provide users with visual feedback on their image and scanned document uploads.
-   **Acceptance Criteria:**
    -   [ ] In the "Uploads" page, the UI displays a thumbnail preview for uploaded images.
    -   [ ] For scanned PDFs, the UI could show the extracted text for the first page as a preview, confirming that OCR was successful.
    -   [ ] When a RAG response cites an image or a scanned document, the frontend displays this clearly (e.g., "From image `chart.png`" or "From `scanned_report.pdf`, page 3").

### 2.4. RAG with Visual Content

-   **Goal:** To enable the chat to seamlessly answer questions using the newly indexed visual content.
-   **Acceptance Criteria:**
    -   [ ] The RAG-enabled chat mode now searches over both text-based and image-based content vectors.
    -   [ ] When a user's query (e.g., "Show me the Q3 sales chart") is embedded, its vector is similar enough to the vector of the AI-generated description ("A bar chart showing a 25% increase in Q3 sales...") to be retrieved as relevant context.
    -   [ ] The chat response correctly uses the retrieved description to answer the user's question.
    -   [ ] The response correctly cites the original image file (e.g., `chart.png`) as the source.

## 3. Definition of Done (DoD)

-   [ ] A user can upload a JPEG image of a bar chart.
-   [ ] The user can then ask, "Show me the document with the Q3 sales bar chart," and the RAG system successfully retrieves and cites the uploaded image.
-   [ ] A user can upload a multi-page, image-only PDF.
-   [ ] The user can ask a specific question about text contained on page 3 of that PDF, and the system provides a correct, cited answer.
-   [ ] The ingestion pipeline correctly routes different file types (text, image, scanned PDF) to the appropriate processing logic.