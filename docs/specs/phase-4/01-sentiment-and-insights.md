# Phase 4 Spec: Sentiment & Insights

**Component:** Backend (`/backend`) & Frontend (`/frontend`)  
**Source Plan:** [`docs/plans/Phased_Plan.md`](/docs/plans/Phased_Plan.md)

## 1. Goal

To leverage a Hugging Face classifier model to perform batch sentiment analysis on unstructured text and to present these findings on a simple, effective insights dashboard in the frontend.

## 2. Feature Breakdown & Acceptance Criteria

### 2.1. Batch Sentiment API

-   **Goal:** An endpoint that can process a large block of text or a collection of text items and return structured sentiment data.
-   **Acceptance Criteria:**
    -   [ ] A new `POST` endpoint is created at `/v1/insights/sentiment/batch`.
    -   [ ] The endpoint accepts a JSON payload containing either a single block of text or an array of text records (e.g., from a CSV upload).
    -   [ ] The backend calls a dedicated Hugging Face classifier model suitable for sentiment analysis.
    -   [ ] **Prompt Engineering:** While not a generative task, the way the data is presented to the classifier is key. The backend prepares the data for the model.
    -   [ ] The results from the model (e.g., per-item sentiment scores) are aggregated.
    -   [ ] The aggregated results, including sentiment distribution and key topics (identified via simple term frequency-inverse document frequency (TF-IDF) or a similar basic algorithm), are stored in the Azure SQL database.
    Example response:
        ```json
        {
          "sentimentDistribution": {
            "positive": 45,
            "negative": 15,
            "neutral": 40
          },
          "trendingTopics": [
            { "topic": "Customer Support", "sentiment": "Negative", "count": 5 },
            { "topic": "Product Quality", "sentiment": "Positive", "count": 12 }
          ]
        }
        ```
    -   [ ] The endpoint returns a job ID or the final analysis results to the client.

### 2.2. Insights Dashboard UI

-   **Goal:** A visual interface for users to submit text for analysis and explore the results.
-   **Acceptance Criteria:**
    -   [ ] A new page/route (e.g., `/insights`) is created in the frontend application.
    -   [ ] The page provides a large text area for users to paste text or a file upload control for a `.csv` or `.json` file containing text records.
    -   [ ] An "Analyze" button triggers the call to the `/v1/insights/sentiment/batch` endpoint.
    -   [ ] While the analysis is processing, the UI displays a loading indicator.
    -   [ ] **Data Visualization:** Upon completion, the dashboard displays:
        -   A **Pie Chart** or **Donut Chart** showing the overall sentiment distribution (e.g., 60% Positive, 25% Negative, 15% Neutral).
        -   A **Data Table** listing the most frequent terms or topics identified in the text, along with their associated sentiment.
    -   [ ] (Stretch Goal) The dashboard includes filters to view results for only positive or negative sentiments.

## 3. Definition of Done (DoD)

-   [ ] A user can paste a block of customer reviews into the insights dashboard and successfully generate a sentiment analysis report.
-   [ ] The resulting charts and tables accurately reflect the sentiment and key topics from the source text.
-   [ ] The process is robust enough to handle a few hundred text records in a single batch request.
-   [ ] The UI provides a clear and intuitive way to understand the analysis results.