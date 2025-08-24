# SQL Best Practices Guide

## Philosophy

* Treat SQL as **first-class code** (version it, review it, test it).
* Model for **access patterns** and integrity first; optimize later with data/metrics.
* Favor **set-based operations** over RBAR (row-by-agonizing-row).
* Keep **transactions short** and **queries explicit** (no `SELECT *`).

## Schema & Modeling

* Normalize to **3NF**; denormalize only for proven hot paths.
* Pick **precise types** (avoid `NVARCHAR(MAX)`/`TEXT` unless needed; use `DECIMAL(p,s)` for money).
* Always define **PKs, FKs, UNIQUE, CHECK** constraints.
* Prefer **surrogate PK** (INT/BIGINT/UUID); add **natural key** as UNIQUE.
* Separate **hot/cold columns** (move large blobs to side tables or Blob storage).
* Design **multi-tenant** by partition key (tenant\_id) or schema-per-tenant.

## Keys, Constraints & Defaults

```sql
-- SQL Server / Postgres style
CREATE TABLE orders (
  order_id      BIGSERIAL PRIMARY KEY,
  tenant_id     INT NOT NULL,
  customer_id   INT NOT NULL,
  status        TEXT NOT NULL CHECK (status IN ('new','paid','shipped','cancelled')),
  total_amount  NUMERIC(12,2) NOT NULL CHECK (total_amount >= 0),
  created_at    TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE (tenant_id, order_id)
);
```

## Indexing Strategy

* Index **foreign keys** and common **filters/sorts**.
* Use **covering indexes** (INCLUDE in SQL Server) for hot queries.
* Prefer **narrow, selective leading columns**; avoid wide composite keys.
* Use **filtered/partial indexes** for sparse data (e.g., soft deletes).
* Periodically **review unused/duplicate** indexes; balance read vs write cost.

```sql
-- SQL Server example
CREATE INDEX IX_orders_tenant_status_created
ON dbo.orders(tenant_id, status, created_at DESC)
INCLUDE(total_amount);

-- Postgres partial index
CREATE INDEX CONCURRENTLY ON orders (created_at DESC)
WHERE status <> 'cancelled';
```

## Query Design Patterns

* **Project explicitly**: `SELECT col1, col2` (avoid `*`).
* **Pagination**: keyset/seek over OFFSET/LIMIT for deep pages.
* Prefer **JOINs** to correlated subqueries; use **window functions** for analytics.
* Avoid **implicit conversions** (match types/lengths) to keep indexes usable.
* Parameterize everything to prevent injection & enable plan reuse.

```sql
-- Keyset pagination (Postgres)
SELECT id, created_at, title
FROM posts
WHERE (created_at, id) < ($1, $2)
ORDER BY created_at DESC, id DESC
LIMIT 50;

-- Window example
SELECT o.customer_id,
       SUM(o.total_amount) AS total,
       RANK() OVER(ORDER BY SUM(o.total_amount) DESC) AS rnk
FROM orders o
GROUP BY o.customer_id;
```

## CTEs, Temp Tables & Upserts

* CTEs improve readability; for reuse within a query, **temp tables** may be faster.
* **Postgres**: `INSERT ... ON CONFLICT DO UPDATE` for upsert.
* **SQL Server**: avoid `MERGE` (footguns); use `UPDATE` then `INSERT` with `NOT EXISTS` in a transaction.

```sql
-- Postgres upsert
INSERT INTO inventory(sku, qty)
VALUES ($1, $2)
ON CONFLICT (sku) DO UPDATE SET qty = inventory.qty + EXCLUDED.qty;

-- SQL Server upsert pattern
BEGIN TRAN;
UPDATE i SET qty = i.qty + @delta FROM dbo.inventory i WHERE i.sku = @sku;
IF @@ROWCOUNT = 0
  INSERT dbo.inventory(sku, qty) VALUES(@sku, @delta);
COMMIT;
```

## Transactions & Concurrency

* Keep transactions **short**; order operations consistently to avoid deadlocks.
* Pick isolation by need: **Read Committed** default; **Snapshot/RC-SI** to reduce blocking; **Serializable** rarely.
* Use **optimistic concurrency** with rowversion/`xmin` and retries for hot rows.

```sql
-- SQL Server optimistic token
rowversion ROWVERSION NOT NULL
```

## Execution Plans & Tuning

* Always inspect plans (`EXPLAIN [ANALYZE]`, SSMS Actual Plan).
* Watch for: table scans on big tables, key lookups, sort/hash spills, implicit converts.
* Match parameter sniffing patterns; consider **OPTIMIZE FOR**, **RECOMPILE** for skewed params (sparingly).
* Maintain **statistics** and **rebuild/reorganize** fragmented indexes during maintenance windows.

```sql
-- Postgres
EXPLAIN (ANALYZE, BUFFERS) SELECT ...;

-- SQL Server
SET STATISTICS IO, TIME ON; -- then run query and read output
```

## Partitioning & Large Tables

* Partition by **time** or **tenant** for prune-able scans and maintenance.
* Keep partitions moderately sized; avoid too many small partitions.
* Use **aligned indexes** per partition; manage sliding windows for retention.

## Security

* Enforce **least privilege** (separate app vs admin roles).
* Always **parameterize**; never concatenate user input.
* Use **encryption at rest** (TDE) and **in transit** (TLS).
* Consider **Row-Level Security** (RLS) for multi-tenant isolation.
* Mask/anonymize PII in non-prod; audit access.

## Maintenance & Reliability

* Backups: follow **3-2-1** (full + diffs/logs), test **restores** regularly.
* Rotate logs; monitor disk growth and autogrowth settings.
* Set **dead letter**/retry patterns at app layer; surface transient errors with retries.

## Testing & Automation

* Version-control **migrations** and review SQL diffs.
* **Unit-test** SQL with tSQLt/pgTAP for critical logic.
* Use **Testcontainers** for integration tests with real engines.
* Generate **idempotent** deployment scripts for multi-envs.

## Anti-Patterns

* `SELECT *` in production queries.
* Over-indexing that cripples write performance.
* Cursors/WHILE loops for bulk ops (use set-based/TVPs/BULK INSERT).
* Business logic buried in triggers.
* Ignoring NULL semantics and collation issues.
* Running without constraints for "performance".

## Quick Checklist

* [ ] Explicit columns, parameterized queries, short transactions.
* [ ] Proper PK/FK/UNIQUE/CHECK; hot queries have covering indexes.
* [ ] Plans inspected; stats up to date; fragmentation under control.
* [ ] Upserts safe; pagination is keyset for deep pages.
* [ ] Security: least privilege, RLS where needed, PII masked in non-prod.
* [ ] Backups tested; migrations idempotent; tests run on real DB in CI.

## References (start here)

* **PostgreSQL**: Query Planning, Indexing, Partitioning chapters.
* **SQL Server**: Execution Plans (Brent Ozar/J. Lewis), Parameter Sniffing, Statistics.
* **General**: Use The Index, Use The Index, Luke; High Performance MySQL (patterns apply broadly).
