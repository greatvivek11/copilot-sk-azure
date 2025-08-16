# Phase 6 Spec: Agents & Tooling

**Component:** Backend (`/backend`) & Frontend (`/frontend`)  
**Source Plan:** [`docs/plans/Phased_Plan.md`](/docs/plans/Phased_Plan.md)

## 1. Goal

To empower the AI assistant with the ability to use "tools." This transforms it from a conversationalist into an agent that can take actions on the user's behalf. This involves creating a set of safe, auditable tools and using a Semantic Kernel planner to orchestrate their use.

## 2. Feature Breakdown & Acceptance Criteria

### 2.1. Tool Plugin Implementation

-   **Goal:** To create a library of well-defined, secure tools that the AI agent can use.
-   **Acceptance Criteria:**
    -   [ ] A new `ToolsPlugin` class is created in the backend project.
    -   [ ] The following native functions are implemented within the plugin, each with a clear `[Description]` attribute explaining its purpose, inputs, and outputs for the planner:
        -   [ ] **`BlobLookupTool`**:
            -   **Description:** "Searches for and retrieves the content of a specific document from Blob Storage."
            -   **Input:** `string documentName`
            -   **Output:** `string documentContent`
        -   [ ] **`SqlReadOnlyQueryTool`**:
            -   **Description:** "Executes a read-only SQL query against the database to answer questions about user or chat metadata."
            -   **Input:** `string sqlQuery`
            -   **Output:** `string queryResultAsJson`
            -   **Security:** This tool MUST use a database user with read-only permissions.
        -   [ ] **`CsvExportTool`**:
            -   **Description:** "Exports a given JSON array into a CSV formatted string."
            -   **Input:** `string jsonData`
            -   **Output:** `string csvContent`

### 2.2. Agentic Chat & Planner Integration

-   **Goal:** To use a planner to interpret a user's complex goal and orchestrate the available tools to achieve it.
-   **Acceptance Criteria:**
    -   [ ] The backend is enhanced with a planner-based execution flow. This could be a new endpoint (e.g., `/v1/agent/execute`) or a mode in the existing chat endpoint.
    -   [ ] When a user provides a complex prompt like, "How many messages were sent yesterday? Export the result as a CSV," the following occurs:
        1.  The planner is invoked with the user's goal.
        2.  The planner is given access to the `ToolsPlugin`.
        3.  The planner generates a multi-step plan, for example:
            -   Step 1: Call `SqlReadOnlyQueryTool` with a query like `SELECT COUNT(*) FROM Messages WHERE...`.
            -   Step 2: Call `CsvExportTool` with the JSON result from Step 1.
        4.  The backend executes this plan step-by-step.
    -   [ ] The final result of the plan (the CSV string) is returned to the user.

### 2.3. Frontend Tool-Use Transparency

-   **Goal:** To provide the user with clear visibility into the agent's actions, building trust and allowing for auditing.
-   **Acceptance Criteria:**
    -   [ ] A new "Agent Mode" or a similar UI is created on the frontend.
    -   [ ] When a user submits a goal to the agent, the UI displays a "timeline" or "step-by-step" view of the plan being executed.
    -   [ ] Each step in the plan is shown with its status: `pending`, `in-progress`, `completed`, or `failed`.
    -   [ ] The UI displays the details of each tool call, including the function name and the (sanitized) input parameters.
    -   [ ] The result of each step is shown before the next one begins.
    -   [ ] The final output of the entire plan is clearly presented to the user.

## 3. Definition of Done (DoD)

-   [ ] A user can ask the agent a question that requires a database lookup (e.g., "How many conversations have I had?").
-   [ ] The frontend UI visualizes the agent's plan: (1) Call `SqlReadOnlyQueryTool`, (2) Display result.
-   [ ] The user receives the correct answer, derived from the tool's execution.
-   [ ] The implementation includes strict security guardrails, especially for the SQL tool, to prevent injection attacks and ensure read-only access.