# EF Core Best Practices Guide

## Philosophy

* **Code First only**; migrations are the single source of truth.
* **Vertical Slice friendly**: inject `DbContext` in handlers; avoid generic repositories unless they add real value.
* **No stored procedures** unless an unavoidable perf edge case (then isolate + document).
* Optimize for **readability, correctness, and profiling-driven performance**.

---

## Setup & Configuration

* Use one `DbContext` per bounded context; keep it **thin**.
* Register `DbContext` with pooling:

```csharp
services.AddDbContextPool<AppDbContext>(o =>
{
    o.UseSqlServer(connString, sql => sql.EnableRetryOnFailure());
    o.EnableSensitiveDataLogging(env.IsDevelopment());
    o.UseQueryTrackingBehavior(QueryTrackingBehavior.NoTracking);
});
```

* Prefer **constructor injection** of `DbContext`.
* Set a **default** tracking behavior to `NoTracking`; opt-in to tracking for commands.

---

## Modeling

* Model **aggregates** explicitly; keep invariants in the aggregate root.
* Use **owned types** for value objects, **backing fields** to protect invariants.
* Keep navigation properties **required** when logically mandatory.
* Avoid bi-directional navigations unless necessary.
* Prefer **records** for immutable read models/DTOs; entities remain mutable within aggregate boundaries.

```csharp
public class Order
{
    private readonly List<OrderLine> _lines = new();
    public int Id { get; private set; }
    public int CustomerId { get; private set; }
    public IReadOnlyList<OrderLine> Lines => _lines;
    public Money Total { get; private set; }

    public void AddLine(Product p, int qty)
    {
        _lines.Add(new OrderLine(p.Id, qty, p.Price));
        Total = Total.Add(p.Price.Multiply(qty));
    }
}

[Owned]
public record Money(decimal Amount, string Currency);
```

---

## Fluent Configurations (no attributes)

* Put mapping in **`IEntityTypeConfiguration<T>`** classes per entity.
* Keep **schema/naming conventions** consistent (snake\_case or PascalCase – choose one).
* Define keys, indexes, and constraints explicitly.

```csharp
public class OrderConfig : IEntityTypeConfiguration<Order>
{
    public void Configure(EntityTypeBuilder<Order> b)
    {
        b.ToTable("orders");
        b.HasKey(x => x.Id);
        b.OwnsOne(x => x.Total);
        b.HasMany(typeof(OrderLine), "_lines").WithOne().OnDelete(DeleteBehavior.Cascade);
        b.HasIndex(x => x.CustomerId);
    }
}
```

---

## Keys & Indexes

* **Surrogate keys** (INT/UUID) for entities; **natural keys** for lookups.
* For SQL Server: prefer **sequential GUIDs** (e.g., Guid v7) to reduce fragmentation.
* Create **composite indexes** in the order of selectivity; use **Include** columns for covering queries.
* Revisit indexes after load tests; remove unused ones.

---

## Migrations

* Generate one migration per feature slice; write **clear migration names**.
* Review SQL with `Script-Migration` (idempotent) and check it in.
* **Apply automatically** only in dev/test; **apply via pipeline** in staging/prod.
* For data moves, write **custom `Up/Down`** steps; avoid raw SQL unless needed.

```bash
# dev
dotnet ef migrations add AddOrderLines --project Infra --startup-project Api
 dotnet ef database update
# prod (pipeline)
dotnet ef migrations script --idempotent -o migrations.sql
```

---

## Querying

* **Project** to DTOs with `.Select(...)` for reads; avoid materializing entire aggregates if not needed.
* Use `AsNoTracking()` for queries that don’t update state.
* Prevent N+1 via `.Include()`/`ThenInclude()` or **explicit projection**.
* Prefer **split queries** for very large graphs: `AsSplitQuery()`.
* Use **Compiled Queries** for hot paths.
* Tag queries to aid tracing: `.TagWith("GetOrdersByCustomer")`.

```csharp
var dto = await db.Orders
    .Where(o => o.CustomerId == id)
    .Select(o => new OrderDto(o.Id, o.Total.Amount))
    .ToListAsync(ct);
```

---

## Change Tracking

* Default to `NoTracking`; enable tracking only for commands.
* For updates, attach a **minimal aggregate**, set state explicitly when needed.
* Avoid long-lived `DbContext`; scope per request/command.

```csharp
// targeted update
var order = new Order { Id = id };
db.Attach(order);
order.AddLine(product, qty);
await db.SaveChangesAsync(ct);
```

---

## Transactions & Concurrency

* One **`SaveChangesAsync` per command**; wrap multiple aggregates in a transaction only when required.
* Use **optimistic concurrency** with a token (`rowversion`/`xmin`/`etag`).
* On conflict, merge or prompt the client to retry.

```csharp
public class Customer
{
    public int Id { get; private set; }
    public string Name { get; private set; } = string.Empty;
    public byte[] RowVersion { get; private set; } = Array.Empty<byte>();
}

public class CustomerConfig : IEntityTypeConfiguration<Customer>
{
    public void Configure(EntityTypeBuilder<Customer> b)
    {
        b.Property(x => x.RowVersion).IsRowVersion();
    }
}
```

---

## Interceptors & Filters

* Use **SaveChanges interceptors** for auditing (CreatedBy/At, ModifiedBy/At).
* Implement **global query filters** for soft delete and multi-tenancy.

```csharp
// Soft delete
b.HasQueryFilter(e => !e.IsDeleted);
```

```csharp
public class AuditInterceptor : SaveChangesInterceptor
{
    public override ValueTask<InterceptionResult<int>> SavingChangesAsync(
        DbContextEventData eventData, InterceptionResult<int> result, CancellationToken ct = default)
    {
        var ctx = eventData.Context!;
        var now = DateTimeOffset.UtcNow;
        foreach (var e in ctx.ChangeTracker.Entries<IAuditable>())
        {
            if (e.State == EntityState.Added) e.Entity.CreatedAt = now;
            if (e.State == EntityState.Modified) e.Entity.ModifiedAt = now;
        }
        return base.SavingChangesAsync(eventData, result, ct);
    }
}
```

---

## Performance

* Inspect generated SQL: `query.ToQueryString()`.
* Batch work; avoid chatty save loops (use **`SaveChanges` once** per command).
* Consider provider-specific bulk libs for large imports.
* Cache reference data with `IMemoryCache` or distributed cache; **invalidate on write**.
* Profile with **MiniProfiler** or similar; use **compiled queries** on hot paths.

---

## Raw SQL & Advanced

* Prefer LINQ. If you must use SQL: `FromSqlInterpolated` with parameters (no string concat).
* **CTEs** are supported via `FromSql` (project to keyless entities if needed).
* Keep raw SQL isolated in a **read model** or dedicated data access type.
* Avoid triggers; if unavoidable (legacy), mirror effects in domain logic and document clearly.

---

## Testing EF Core

* Use **Testcontainers** with the real provider (SQL Server/Postgres) for fidelity.
* SQLite in-memory is fine for **pure LINQ** tests but beware of provider differences.
* Seed data per test; ensure isolation and cleanup.
* Verify SQL with snapshot tests on `ToQueryString()` for critical queries.

---

## Telemetry & Diagnostics

* Log command execution time and row counts; tag with **correlation IDs**.
* Emit metrics: query latency p95/p99, error rate, open transactions, deadlocks.

---

## Anti-Patterns to Avoid

* Generic repository that hides EF Core features and encourages anemic models.
* Long-lived `DbContext` shared across threads.
* Loading entire graphs when only projections are needed.
* Multiple `SaveChanges` calls inside loops.
* Silent swallowing of `DbUpdateConcurrencyException`.

---

## Compiled Queries (example)

* Use `EF.CompileQuery` for CPU-bound, hot query paths to avoid repeated expression compilation overhead.

```csharp
// static compiled query
private static readonly Func<AppDbContext, int, CancellationToken, Task<OrderDto?>> _getOrderById
    = EF.CompileAsyncQuery((AppDbContext ctx, int id, CancellationToken ct) =>
        ctx.Orders
           .Where(o => o.Id == id)
           .Select(o => new OrderDto(o.Id, o.Total.Amount))
           .FirstOrDefault());

// Usage
var dto = await _getOrderById(db, id, ct);
```

---

## Example Migration with Data Move (custom Up/Down)

* When evolving schema and migrating data, write explicit `Up` and `Down` steps that are idempotent and reversible when possible.

```csharp
public partial class SplitFullName : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.AddColumn<string>("FirstName", "users", nullable: true);
        migrationBuilder.AddColumn<string>("LastName", "users", nullable: true);

        // Data move - use SQL with parameters; keep it reviewable
        migrationBuilder.Sql(@"
            UPDATE users
            SET FirstName = SUBSTRING(FullName, 1, CHARINDEX(' ', FullName) - 1),
                LastName = SUBSTRING(FullName, CHARINDEX(' ', FullName) + 1, 200)
            WHERE FullName IS NOT NULL
        ");

        migrationBuilder.AlterColumn<string>("FirstName", "users", nullable: false);
        migrationBuilder.AlterColumn<string>("LastName", "users", nullable: false);
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        // reverse the change
        migrationBuilder.AddColumn<string>("FullName", "users", nullable: true);
        migrationBuilder.Sql(@"
            UPDATE users
            SET FullName = FirstName + ' ' + LastName
            WHERE FirstName IS NOT NULL OR LastName IS NOT NULL
        ");
        migrationBuilder.DropColumn("FirstName", "users");
        migrationBuilder.DropColumn("LastName", "users");
    }
}
```

**Guidance:** always review the generated SQL, ensure migrations are idempotent, and consider locks/transaction size when moving large datasets (batch in steps if needed).

---

## Bulk Import Strategy (provider-agnostic)

* For large imports/ETL, avoid row-by-row `Add` + `SaveChanges` loops. Consider these patterns:

  * **Batching**: accumulate N entities then call `SaveChanges` in a single transaction.
  * **Bulk libraries**: when available (EFCore.BulkExtensions, Z.BulkOperations) for SQL Server/Postgres.
  * **Staging table**: bulk-copy into a staging table (e.g., SqlBulkCopy for SQL Server), then run set-based SQL to merge into target tables.
  * **Change detection off**: temporarily disable `AutoDetectChangesEnabled` during bulk inserts.

```csharp
// Example batching
ctx.ChangeTracker.AutoDetectChangesEnabled = false;
int batchSize = 1000;
for (int i = 0; i < items.Count; i += batchSize)
{
    var batch = items.Skip(i).Take(batchSize);
    ctx.AddRange(batch);
    await ctx.SaveChangesAsync(ct);
    ctx.ChangeTracker.Clear();
}
ctx.ChangeTracker.AutoDetectChangesEnabled = true;
```

**Guidance:** measure RU/throughput; choose staging+set-based merges for transformational workloads. Use provider-specific bulk-copy when throughput matters.

---

## Cosmos-specific Patterns (EF Core vs SDK)

* Cosmos provider in EF Core is convenient for simple scenarios but has limitations (transactions, SQL dialect, RU control, change feed features). For advanced scenarios prefer the Cosmos SDK for:

  * Fine-grained RU management and diagnostics.
  * Change Feed processing with explicit leases (high-performance CDC-like flows).
  * Bulk executor and transactional batch support tuned per partition key.
  * Complex queries and server-side pagination patterns.

**When to use EF Core with Cosmos:**

* Small projects or when you want parity with relational code.
* Simple CRUD where EF abstractions map well to Cosmos semantics.

**When to use Cosmos SDK directly:**

* You need advanced RU tuning, bulk operations, or change feed processing.
* When multi-document transactions per partition or cross-partition optimizations are required.

**Tips:**

* Design partition keys for your query/write patterns; test with realistic partition sizes.
* Store vector embeddings and large blobs in Blob Storage or a dedicated vector store; store metadata and pointers in Cosmos to avoid large RU costs.
* Keep item sizes modest; large documents increase RU and latency.

---

## Provider Notes

* **SQL Server**: use `EnableRetryOnFailure`, `datetime2`, `rowversion`, sequential GUIDs.
* **Cosmos**: EF Core provider has feature gaps; for advanced scenarios (vectors, RU tuning), consider the **Cosmos SDK** directly for read models.

---

## Checklist (TL;DR)

* [ ] Code First only; migrations in repo, applied via pipeline.
* [ ] Default `NoTracking`; project to DTOs for reads.
* [ ] One `SaveChanges` per command; use optimistic concurrency.
* [ ] Interceptors for audit; filters for soft delete/tenancy.
* [ ] Indexes reviewed after load tests; compiled queries for hot paths.
* [ ] Test with real DB via Testcontainers.
