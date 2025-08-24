# EF Core + Data Modeling & Performance Best Practices Guide

## Philosophy

* Use EF Core as an **ORM, not a replacement for SQL**.
* Let your **domain model drive design**, not the database.
* Embrace **Code First with migrations** for long-term maintainability.
* Keep EF Core configuration close to the model (via **EntityTypeConfiguration classes**).
* Write LINQ that translates well to SQL; avoid **client-side evaluation**.
* Design with **performance and scalability** in mind: model first, then optimize queries and persistence.

## DbContext Configuration

* Use **`AddDbContextPool`** for efficient DbContext management.
* Enable **retry on failure** for transient faults (`EnableRetryOnFailure`).
* Default to **NoTracking** queries; opt-in to tracking only when needed.
* Configure **logging and sensitive data logging** in non-production only.

```csharp
services.AddDbContextPool<AppDbContext>(options =>
    options.UseSqlServer(connStr, sql => sql.EnableRetryOnFailure())
           .UseQueryTrackingBehavior(QueryTrackingBehavior.NoTracking));
```

## Data Modeling

* Normalize until it hurts, denormalize until it works.
* Model **aggregates and boundaries** explicitly; don’t let EF dictate domain design.
* Prefer **value objects** for concepts like Money, Email, Address.
* Use **enums with value converters** (never raw integers).
* For large-scale systems: partition data by tenant, region, or domain.
* Use **composite keys** only when natural, otherwise prefer surrogate keys (GUID, int).
* Use **owned types** for immutability and encapsulation.
* Balance depth: avoid very deep navigation graphs.
* For **multi-tenant systems**, consider schema-per-tenant or partition-per-tenant strategies.

### Partitioning Example

```sql
-- Sharding strategy (per region)
CREATE TABLE Orders_EU (...);
CREATE TABLE Orders_US (...);
```

## EntityTypeConfiguration

* Keep model configuration in **separate classes** via `IEntityTypeConfiguration<T>`.
* This avoids clutter in `DbContext.OnModelCreating`.

```csharp
public class OrderConfig : IEntityTypeConfiguration<Order>
{
    public void Configure(EntityTypeBuilder<Order> builder)
    {
        builder.OwnsOne(o => o.Total);
        builder.HasMany(o => o.Lines).WithOne();
    }
}
```

## Queries & Performance

* Use **projections** (`Select`) instead of fetching entire entities.
* Prefer **split queries** for large object graphs (`AsSplitQuery`).
* Use **compiled queries** for hot paths.
* Avoid `Include` everywhere — load only what’s needed.
* Always inspect **`ToQueryString()`** for critical queries.

```csharp
var dto = await context.Orders
    .Where(o => o.Id == id)
    .Select(o => new OrderDto(o.Id, o.Total.Amount))
    .FirstAsync();
```

### N+1 Anti-pattern Example

```csharp
// BAD: Triggers multiple queries
var orders = await context.Orders.ToListAsync();
foreach (var o in orders)
    Console.WriteLine(o.Customer.Name);

// GOOD: Projection fetches in single query
var orders = await context.Orders
    .Select(o => new { o.Id, CustomerName = o.Customer.Name })
    .ToListAsync();
```

## Indexing & Keys

* Define indexes explicitly using `HasIndex`.
* Always index foreign keys.
* Use **filtered indexes** for soft deletes.
* For Cosmos DB, always design for **partition keys** upfront.

```csharp
builder.HasIndex(o => o.CustomerId);
builder.HasIndex(o => new { o.Status }).HasFilter("[IsDeleted] = 0");
```

## Migrations

* Use **migrations for schema evolution**; don’t drop & recreate in production.
* Write **custom `Up`/`Down`** when moving or seeding data.
* Review generated SQL before applying in production.
* Use **idempotent scripts** for multi-environment deployments.

```csharp
protected override void Up(MigrationBuilder migrationBuilder)
{
    migrationBuilder.Sql("UPDATE Orders SET Status = 'Archived' WHERE IsDeleted = 1");
}
```

## Concurrency

* Use **optimistic concurrency** with `rowversion`/`timestamp`.
* Handle `DbUpdateConcurrencyException` explicitly.
* Prefer **last-write-wins** only for non-critical fields.

```csharp
public class Order
{
    public int Id { get; set; }
    public byte[] RowVersion { get; set; } = default!;
}
```

## Bulk Operations

* EF Core is not optimized for bulk insert/update.
* Strategies:

  * Batch in chunks with `AddRangeAsync` + `SaveChanges`.
  * For large datasets, prefer **provider-specific bulk tools** (e.g., `SqlBulkCopy`).
  * Use **raw SQL** for massive imports.
  * Consider **staging tables** for ETL.

```csharp
await context.BulkInsertAsync(entities); // if using EFCore.BulkExtensions
```

## Advanced SQL Features

* **Stored procedures**: use only when performance demands.
* **Functions**: map scalar/table functions for complex logic.
* **CTEs**: use `FromSqlInterpolated` for recursive or analytical queries.

```csharp
var data = context.Users
    .FromSqlInterpolated($@"
        WITH RecursiveCTE AS (...)
        SELECT * FROM RecursiveCTE");
```

## Caching & Performance Tuning

* Use **second-level caching libraries** (e.g., EFCoreSecondLevelCacheInterceptor).
* Use **compiled queries** for hot paths.
* Disable **AutoDetectChanges** in bulk operations.
* Always measure with **SQL Profiler / Application Insights**.

## Triggers & Interceptors

* Avoid database triggers unless legacy system demands.
* Use EF Core **interceptors** for auditing, soft deletes, multi-tenancy.

```csharp
public class AuditInterceptor : SaveChangesInterceptor
{
    public override int SavedChanges(SaveChangesCompletedEventData eventData, int result)
    {
        // add audit log
        return base.SavedChanges(eventData, result);
    }
}
```

## Testing

* For unit tests: mock DbSets with in-memory collections.
* For integration tests: use **SQLite in-memory** or **Testcontainers**.
* Always validate generated SQL with `ToQueryString()`.

```csharp
var sql = context.Orders.Where(o => o.Id == 1).ToQueryString();
```

### CQRS Testing Example

* Query side can be validated by projections.
* Command side can be tested with event sourcing or audit logs.

## Cosmos DB with EF Core

* EF Core provider is limited:

  * No joins across containers.
  * Limited LINQ translation.
* Prefer **Cosmos SDK** for full flexibility (bulk ops, transactional batch).
* EF Core Cosmos is useful for **simple aggregates** when you want consistency with relational code.

## Data Modeling Anti-Patterns

* Treating EF as a **black box**; always know the SQL generated.
* Over-fetching with `Include` everywhere.
* Over-normalizing at the cost of performance.
* Catching `Exception` instead of `DbUpdateException`/`DbUpdateConcurrencyException`.
* Ignoring **N+1 query issues**.
* Using EF migrations to seed huge data sets.
* Designing without considering **indexing strategy**.
* Using surrogate keys **without natural uniqueness constraints**.

## SQL Tuning Recipes (Appendix)

* Use **EXPLAIN / Execution Plans** to detect table scans.
* Add missing indexes for slow queries.
* Replace `SELECT *` with explicit projections.
* Avoid `IN` with large sets — use joins or temp tables.
* Batch writes instead of row-by-row inserts.
