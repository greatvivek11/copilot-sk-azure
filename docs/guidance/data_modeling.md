# Data Modeling & Performance — Best Practices

## Philosophy

* Model for **access patterns** first: reads and writes drive structure.
* Optimize for **correctness, observability, and evolvability** before micro-optimizing for perf.
* Choose the right **data store for the problem** (relational vs document vs vector vs time-series).
* Prefer **event-driven, idempotent writes** and clear consistency contracts.

## High-Level Design Principles

* Start with **workflows & queries** → derive write model → derive physical schema.
* Apply **single responsibility** for tables/containers: one concern per store.
* Keep **aggregation boundaries** explicit; design for transactional boundaries (aggregate root).
* Use **domain events** for cross-cutting consistency and async denormalization.

## Relational Modeling (OLTP)

* Use **normalized schema** for transactional integrity; denormalize for read performance when necessary.
* Prefer **surrogate keys** (INT/UUID) for PKs; include natural keys as unique constraints if useful.
* Design **FKs** for integrity; use cascades deliberately and document rationale.
* Use **narrow, covering indexes**: leading column = highest selectivity; include extra columns to avoid lookups.
* Use **partial/filtered indexes** for sparse patterns.
* Avoid wide rows; factor large or infrequently-read columns into separate tables.

**Example: composite index**

```sql
CREATE INDEX IX_orders_customer_status ON orders(customer_id, status) INCLUDE(total_amount, created_at);
```

## Read Models & CQRS

* Create **read-optimized views/materialized views** for heavy reporting paths.
* Maintain **denormalized read models** asynchronously (change data capture, domain events) to keep writes fast.
* Use **materialized views** or dedicated read stores (Elasticsearch, Redis) for search and fast queries.
* Keep read models **eventually consistent** and document acceptable staleness.

## Document Stores (Cosmos, MongoDB)

* Design by **query & partition key**: choose partition key with even cardinality and aligned with queries.
* Embed 1-to-few relationships; reference for 1-to-many with fan-out.
* Keep documents **size-bounded** (avoid very large documents). Split into chunks if necessary.
* Use **TTL** for ephemeral data; leverage change feed for downstream processing.

**Tip:** store large binary blobs in Blob Storage and reference them from documents.

## Vector Data & Semantic Search

* Store embeddings in a dedicated vector store or a DB with vector index support.
* Co-locate metadata for filtering; keep embeddings as fixed-length float arrays and record model/version.
* Chunk large documents (e.g., 500–1,000 tokens) with overlap and store chunk offsets for provenance.
* Record embedding model name and vector dimension in metadata to allow safe reindexing.

**Example structure (pseudo-JSON)**

```json
{
  "id": "doc-1#chunk-3",
  "vector": [0.12, -0.03, ...],
  "sourceId": "doc-1",
  "chunkIndex": 3,
  "text": "...",
  "embeddingModel": "text-embed-xyz/2025-01"
}
```

## Indexing Strategies

* Index according to **filter, order, and join** patterns.
* For relational DBs, prefer **covering indexes** for hot queries to avoid lookups.
* For document stores, design **composite and time-series indexes** to support range scans and recent-first queries.
* Regularly **review index usage** and drop unused indexes (they cost writes and space).

## Partitioning & Sharding

* Partition large tables by **time** or logical tenant to improve maintenance and pruning.
* Shard when a single node cannot handle RU/IOPS or storage; choose shard key with good cardinality.
* Plan resharding strategy early (e.g., tenant-aware sharding) — resharding is hard.

## Time-Series & Telemetry Data

* Use specialized TSDBs (Influx, Timescale) where possible; if using relational, bucket by day/month.
* Store raw events in append-only tables and maintain rollups for long-term queries.
* Apply TTL/retention policies and automated archiving to cold storage (Blob/Archive tiers).

## OLAP & Analytics

* Use columnar stores or a dedicated warehouse (Synapse, BigQuery) for heavy analytic workloads.
* ETL/ELT: prefer ELT where possible (push transformations to the warehouse for scale).
* For batch workloads, use incremental loads and watermarking to avoid full scans.

## Query Performance & Diagnostics

* Use **EXPLAIN / Query Plan** to understand index usage and scans vs seeks.
* Monitor **wait stats, CPU, IO, memory, and plan cache** on RDBMS.
* Track **p95/p99 latency** and correlate with query text and parameter patterns.
* Enable **slow query logging** and alert on regressions.

**Quick diagnostic SQL (Postgres)**

```sql
EXPLAIN (ANALYZE, BUFFERS) SELECT ...;
```

## Schema Evolution & Migrations

* Maintain migrations in source control; prefer **incremental, small migrations**.
* For large data moves, perform **online migrations** with backfill jobs and feature flags:

  1. Deploy schema additions (new columns/tables) nullable.
  2. Backfill in the background.
  3. Flip reads to new model.
  4. Make columns non-nullable and remove old fields.
* Avoid long blocking operations; use batching for transforms.

## Caching Strategies

* Cache at multiple levels: CDN (for static), API layer (response cache), object cache (Redis) for hot keys.
* Cache invalidation: prefer **explicit invalidation** on writes or short TTLs for eventually consistent data.
* Use **cache-aside** pattern and guard against thundering herds with locks or request coalescing.

## Concurrency & Consistency

* Prefer **optimistic concurrency** in OLTP systems; implement `rowversion`/ETags.
* Use **idempotency keys** for external-facing write endpoints to prevent duplicate processing.
* When strong consistency is required across services, use **distributed transactions sparingly**; prefer saga patterns for long-running workflows.

## Data Integrity & Governance

* Enforce constraints at the DB level where possible (unique, fk, check constraints).
* Apply **data classification** and retention policies for GDPR/compliance.
* Audit sensitive changes (who/when) with immutable audit logs.
* Encrypt PII at rest and in transit; mask or tokenize sensitive values in logs.

## Bulk Import & ETL Patterns

* For large ingests, use **staging tables** + set-based operations for merges.
* Use streaming ingestion (Kafka/Event Hubs) for continuous loads; batch for cost-efficiency.
* Monitor throughput, backpressure, and implement dead-letter queues for problematic records.

## Cost & RU Optimization (Cosmos / Cloud DBs)

* For RU-based systems (Cosmos): optimize item size, partition key, and query filters to reduce RU cost.
* Use **server-side pagination** and limit projections to necessary fields.
* Monitor 429s and set autoscale or appropriate RU/s limits.

## Observability & SLAs

* Emit data-layer metrics: query latency, rows returned, RU consumed, deadlocks, failed queries.
* Tag metrics with service, operation, and environment.
* Establish SLOs for p95/p99 and alert on degradation.

## Security Considerations

* Principle of least privilege for DB credentials and service accounts.
* Use Managed Identities and short-lived credentials where supported.
* Avoid putting secrets in code or config; rotate keys regularly.

## Checklist (TL;DR)

* [ ] Model by access patterns; design aggregates and boundaries.
* [ ] Index for filters and ordering; review after load tests.
* [ ] Partition/shard thoughtfully; plan resharding.
* [ ] Use read models for heavy reporting/search.
* [ ] Track and optimize slow queries with query plans.
* [ ] Cache responsibly and handle invalidation.
* [ ] Design for idempotency and eventual consistency where applicable.
* [ ] Enforce governance: encryption, retention, audits.

## Further Reading (suggested)

* Database vendor docs: Postgres, SQL Server, Cosmos DB.
* Design patterns: CQRS, Saga, Event Sourcing (use only when justified).
* Observability: OpenTelemetry, Prometheus, Application Insights.
