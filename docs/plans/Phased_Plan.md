# Phased Implementation Plan

This plan is intentionally **high-level but actionable**. It defines scope, milestones, acceptance criteria (DoD), and risks for each phase. It is the only place where phases are maintained. Other docs should link here.

> Constraints: PoC/resume project, cost-sensitive (avoid paid services like Private Endpoints, Key Vault, Defender for Cloud). Azure Container Apps (ACA) + Dapr, Azure SQL, Blob Storage, Cosmos DB (vector), Semantic Kernel (SK) + Hugging Face Inference Router via OpenAI connector. Frontend: React 19 + Vite + Tailwind v4. Backend: .NET 10 Minimal APIs, Vertical Slice Architecture, **Mediator (source-gen) instead of MediatR**.

---

## Phase 0 — Cloud Foundation & Scaffolding

**Goals**

* Establish repo, CI/CD, infra, Dapr service invocation, and SK→HF connectivity.

**Scope (In)**

* Monorepo structure, Dockerfiles, base GitHub Actions.
* Provision: GHCR, ACA env, ACA apps (FE public, BE internal), Azure SQL, Blob, Cosmos (vector) — all minimal SKUs/free tiers where possible.
* Dapr enabled for FE and BE; FE invokes BE via Dapr app-id.
* Secrets via GitHub Actions; Managed Identity planned later.
* Health endpoints and smoke tests.

**Scope (Out)**

* Private Endpoints, Key Vault, Defender for Cloud (documented as optional).

**Key Tasks**

* Repo scaffold: `/frontend`, `/backend`, `/infra`, `.github/workflows`.
* Dockerize FE/BE; local dev with `docker compose` and Dapr sidecars.
* Bicep/Terraform: create ACA env + apps, GHCR, SQL (basic), Blob, Cosmos (Mongo vCore + vector).
* Configure FE public ingress + custom domain (optional), BE internal.
* Add `/v1/health` and kernel bootstrap (SK + HF Router via OpenAI connector).
* FE “Status” page invoking `/v1/health` through Dapr.

**Definition of Done (DoD)**

* CI builds images and pushes to GHCR on main branch.
* ACA deploys FE+BE; FE→Dapr→BE `/v1/health` returns 200.
* Basic logs visible (console / ACA logs).

**Risks & Mitigations**

* Model/connectivity issues → add fallback “echo” handler.
* ACA/Dapr config drift → encode in IaC and env variables with defaults.

---

## Phase 1 — Core Chat Assistant

**Goals**

* End-to-end chat with streaming, persisted sessions/messages, and telemetry.

**Scope (In)**

* `/v1/chat` endpoint using SK with HF Router.
* Streaming responses to FE (SSE or chunked fetch).
* EF Core + Azure SQL for sessions/messages.
* **Mediator (source-gen)** for lightweight CQRS per slice (or keep pure VSA if preferred).

**Key Tasks**

* Domain: `User`, `Conversation`, `Message` entities; migrations.
* Feature slice: `Chat/` with `Endpoint`, `Command/Query`, `Handlers`.
* FE chat UI: session switcher, message list, input, “regenerate”, error states.
* Observability: correlation id from FE, request/response meta (PII-safe).

**DoD**

* Multiple concurrent sessions per user.
* Messages persisted; reload restores history.
* Streaming works smoothly; basic rate limiting/backoff.

**Risks**

* HF Router rate limits → add exponential backoff + retry.

---

## Phase 2 — Document Ingestion & RAG

**Goals**

* Upload → extract → chunk → embed → vector store; Q\&A over docs with citations.

**Scope (In)**

* Signed upload to Blob; metadata row in SQL.
* Extraction: text from PDF/DOCX/TXT (OSS libs), basic cleanup.
* Chunking (token-aware) + embeddings via HF models.
* Cosmos DB vector upsert; retrieval (top-k) with citation metadata.
* Chat mode toggle: “Ask about docs”.

**Key Tasks**

* API: `/v1/uploads/signed-url`, `/v1/ingest`.
* Worker (simple in-process or ACA Job) for extraction & embedding.
* Data model: `Document`, `Chunk` (id, docId, span, vector, sourceUri, page).
* FE: uploads page with progress; Q\&A mode with source chips.

**DoD**

* Upload doc; ask a question; see grounded answer with 2–5 citations.
* Cosmos vector index populated; latency acceptable for PoC.

**Risks**

* Large PDFs → cap file size and page count; background job for long runs.

---

## Phase 3 — Vision & OCR

**Goals**

* Image analysis and OCR for image-only PDFs; index extracted text for RAG.

**Scope (In)**

* Image ingestion to Blob; OCR (Tesseract/PaddleOCR container) or HF vision model.
* Extracted text piped into Phase-2 chunking/embedding flow.
* FE shows image preview + extracted text snippet.

**DoD**

* Upload image/scanned PDF; ask about its content; see citations referencing image/page.

**Risks**

* OCR accuracy on complex docs → allow manual correction or re-OCR option.

---

## Phase 4 — Sentiment & Insights

**Goals**

* Batch sentiment/classification and a small insights dashboard.

**Scope (In)**

* `/v1/sentiment/batch` endpoint; HF classifier model.
* Store per-item scores + aggregates in SQL.
* FE dashboard with trend, distribution, top terms (simple TF-IDF).

**DoD**

* Upload CSV/JSON of texts; run analysis; view charts with filters.

**Risks**

* Model bias/noise → show confidence and allow thresholding.

---

## Phase 5 — Long-Term Memory

**Goals**

* Summarize and store conversation memories; personalize retrieval context.

**Scope (In)**

* Scheduled summary job per conversation; memory vectors in Cosmos.
* On session start, load N relevant memory snippets into system prompt.

**DoD**

* Returning to a session influences assistant with prior context (visible to user).

**Risks**

* Prompt bloat → cap memory items; summarize aggressively.

---

## Phase 6 — Agents & Tooling

**Goals**

* Tool-augmented assistant with a few safe, auditable tools.

**Scope (In)**

* SK tools: `blob.lookup`, `sql.readonly.query`, `csv.export`.
* Planner or routing logic for when to call tools.
* FE timeline displays tool calls and results for transparency.

**DoD**

* Assistant can decide to fetch a document or run a read-only SQL query, showing the step in UI.

**Risks**

* Tool abuse → strict input validation; read-only SQL user; guardrails in prompts.

---

## Phase 7 — Hardening & Optimization

**Goals**

* Enterprise polish while staying cost-aware.

**Scope (In)**

* Entra ID authN/Z (optional for PoC); roles (Uploader, Analyst).
* Managed Identity to Blob/Cosmos/SQL (if feasible in ACA for PoC).
* Caching and cost controls (response length, temperature, chunk size).

**DoD**

* Security model documented; costs/limits documented; load test baseline captured.

**Risks**

* Over-engineering → keep optional features disabled by default.

---

## Cross-Cutting: Testing & Quality

* **Unit**: handlers, services, chunking, embeddings adapter.
* **Integration**: FE→Dapr→BE; SQL/Blob/Cosmos adapters.
* **Smoke/Canary**: health checks; simple chat and upload flows post-deploy.
* **AI Eval**: prompt regression examples; RAG hit-rate sampling.

---

## Project Ops

* **GitHub Labels**: `phase:p0`..`phase:p7`, `area:fe`, `area:be`, `infra`, `ai`, `docs`, `good-first-issue`.
* **Milestones**: P0..P7 with DoD copied from above.
* **Docs**: Each phase adds/updates a short `DECISIONS.md` entry.
* **Cost Guardrails**: log monthly ingestion to stay under free quotas; sampling on by default.
