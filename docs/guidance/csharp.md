# C# Best Practices Guide

## Philosophy

* Write **modular, simple, functional** code.
* Favor **readability and maintainability** over cleverness.
* **Composition over inheritance**.
* **Low coupling, high cohesion**.
* Apply **SOLID principles** pragmatically.

---

## Language Features & Syntax

* Use **expression-bodied members** for brevity when appropriate.
* Prefer **`var`** when the type is obvious; otherwise be explicit.
* Use **`record`** for immutable data containers; classes for behavior.
* Default to **immutable types**: `readonly` fields, `init` setters, `readonly struct`.
* Prefer **pattern matching** (`switch`, `is`, `when`) over nested `if/else`.
* Use **nullable reference types** (`#nullable enable`) and treat warnings as errors.
* **Avoid default interface methods**; prefer clean abstractions and dedicated extension methods.

```csharp
// Example: record for immutable DTO
public record UserDto(string Id, string Name);
```

---

## Error Handling & Result Pattern

* Fail fast with **guard clauses**.
* Use **exceptions for exceptional cases only**; not for flow control.
* Wrap external calls with **resilience policies** (retry, timeout, circuit breaker).
* Always **log exceptions** with context, but avoid duplicate logging.
* Prefer **custom exception types** for domain errors.
* For predictable errors, use a **Result\<TSuccess, TFailure> pattern** instead of exceptions.

```csharp
public record Result<TSuccess, TFailure>(TSuccess? Success, TFailure? Failure)
{
    public bool IsSuccess => Failure is null;
    public static Result<TSuccess, TFailure> Ok(TSuccess value) => new(value, default);
    public static Result<TSuccess, TFailure> Fail(TFailure error) => new(default, error);
}

// Usage
var result = ValidateUser(input);
if (!result.IsSuccess)
    return Results.BadRequest(result.Failure);
```

---

## LINQ & Collections

* Prefer **LINQ** for declarative data transformations.
* Avoid multiple enumerations; materialize with `.ToList()` or `.ToArray()` when needed.
* Use **`IReadOnlyCollection<T>`/`IEnumerable<T>`** in APIs; expose minimal surface.
* Beware of **LINQ-to-Objects vs LINQ-to-Entities** differences.
* Use **`Span<T>`/`Memory<T>`** for high-performance scenarios.

---

## Async & Concurrency

* **Async all the way**: never block with `.Result` or `.Wait()`.
* Pass **`CancellationToken`** through all async methods.
* Use **`ValueTask`** sparingly (only for hot paths).
* Prefer **channels** or **TPL Dataflow** for producer/consumer pipelines.
* Avoid shared mutable state; use **immutable messages**.
* Use **IAsyncEnumerable<T>** for async streaming instead of buffering everything.

---

## Object-Oriented & Functional Style

* Keep **entities small** and focused; prefer **value objects**.
* **Encapsulate invariants** inside constructors/factories.
* Use **extension methods** for functional utilities and fluent chaining.
* Favor **pure functions**; isolate side effects at boundaries.
* Use **records + pattern matching** for discriminated unions (e.g., Result<T>).

```csharp
public static class StringExtensions
{
    public static string ToSnakeCase(this string input)
    {
        // simple example
        return string.Concat(input.Select((c, i) => i > 0 && char.IsUpper(c)
            ? "_" + c
            : c.ToString())).ToLower();
    }
}

// Usage
var snake = "HelloWorld".ToSnakeCase(); // "hello_world"
```

---

## Structs vs Records

* Use **records** for immutable reference types with equality.
* Use **structs** only for:

  * Small, immutable, value-type semantics.
  * High-performance scenarios to avoid allocations.
  * When instances are frequently created/discarded.
* Use **readonly struct** for safety and clarity.

---

## File Organization

* Break large files into **partial classes** to separate concerns (esp. with generated code).
* Group related methods in **static classes** when they are stateless utilities.
* Avoid dumping everything into one helper class; organize by domain.
* Keep namespaces aligned with folder structure.

---

## Function Chaining & Fluent Style

* Use **extension methods** to enable fluent pipelines.
* Chain pure transformations for clarity.

```csharp
var processed = input
    .Trim()
    .ToLower()
    .ToSnakeCase();
```

---

## Recommended Patterns & Practices

* **Result\<T, E>**: Functional error handling without exceptions.
* **Maybe/Option**: Represent missing values explicitly.
* **Guard Clauses**: Fail fast at method entry.
* **Pipeline/Function Chaining**: Compose logic fluently.
* **Null Object Pattern**: Avoid null checks with safe defaults.
* **Immutability**: Prefer `readonly`/`init` and defensive copying.
* **Analyzers**: Use Roslyn analyzers, StyleCop, Sonar for consistency.
* **Async Streams**: Use `IAsyncEnumerable<T>` for streaming results instead of buffering.

---

## Coding Standards

* Follow **.NET naming conventions** (PascalCase for public, camelCase for locals/fields).
* Keep methods **< 30 lines**; extract private methods for clarity.
* Limit classes to **one responsibility**.
* Avoid magic numbers/strings: use **constants** or **enums**.
* Document only when intent isn’t obvious; prefer **self-explanatory code**.
* Use **Roslyn analyzers, StyleCop, or Sonar** for automated static analysis.

---

## Performance

* Benchmark with **BenchmarkDotNet** before optimizing.
* Use **`Span<T>`**, `Memory<T>`, and **`ref struct`** for hot paths.
* Avoid unnecessary allocations; pool objects with `ArrayPool<T>` when needed.
* Use **`ConfigureAwait(false)`** in library code.
* Be mindful of **boxing/unboxing** with generics and value types.
* Apply **immutability practices** (defensive copying) to prevent hidden bugs.

---

## Testing

* Favor **unit tests for pure functions**, **integration tests** for behaviors.
* Test public contracts, not private implementation details.
* Use **xUnit** with fluent assertions.
* Apply **parameterized tests** for broader coverage.
* Mock only external dependencies; prefer **in-memory fakes**.

---

## Anti-Patterns to Avoid

* God objects / large service classes.
* Excessive inheritance hierarchies.
* Static state and singletons (unless stateless + thread-safe).
* Catching general exceptions (`catch (Exception)`) without handling.
* Overusing reflection and dynamic for regular scenarios.
* Default interface methods as a substitute for clean abstractions.
