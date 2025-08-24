# .NET Best Practices Guide

## Philosophy

* Treat .NET as a **platform** – focus on runtime, libraries, tooling, deployment.
* Build applications to be **resilient, observable, secure, and cloud-ready**.
* Follow **Microsoft's .NET and Azure Well-Architected Frameworks** for alignment.
* Optimize for **developer productivity** without sacrificing **runtime efficiency**.

---

## Application Structure

* Use **minimal APIs** for lightweight services; MVC for complex apps.
* Organize projects by **bounded contexts or vertical slices**.
* Apply **Dependency Injection** everywhere – leverage .NET built-in DI.
* Prefer **configuration via appsettings.json** and environment variables.
* Centralize **logging and telemetry** at startup.

---

## Configuration & Options

* Use **Options pattern** (`IOptions<T>`, `IOptionsSnapshot<T>`) for config.
* Bind strongly-typed config classes instead of magic strings.
* Secure secrets with **Managed Identity** instead of raw Key Vault lookups.
* Support **multiple environments** via `ASPNETCORE_ENVIRONMENT`.

---

## Error Handling & Resilience

* Use **global error handling middleware**.
* Standardize **problem details (RFC 7807)** responses for APIs.
* Wrap external calls with **Polly** (retry, circuit breaker, fallback).
* Prefer **Result pattern** for domain logic errors.

---

## Data & Persistence

* Use **EF Core** for ORM, but abstract with Repository + Unit of Work if domain-heavy.
* Apply **migrations** for schema evolution.
* For high-performance workloads, consider **Dapper** or raw ADO.NET.
* Prefer **asynchronous EF Core APIs**.
* Use **caching** (MemoryCache, Distributed Cache, Redis) strategically.

---

## API Design

* Stick to **RESTful principles** for APIs.
* Use **versioning** (URL or header based).
* Always enable **OpenAPI/Swagger** for contracts.
* Prefer **ProblemDetails** for errors.
* Apply **rate limiting & throttling** middleware.

---

## Security

* Always use **HTTPS**.
* Apply **ASP.NET Core Identity** or external providers (Azure AD, IdentityServer).
* Leverage **role-based (RBAC) and policy-based authorization**.
* Avoid storing secrets; use **Managed Identity** where possible.
* Apply **Data Protection API** for encryption at rest.
* Validate inputs with **FluentValidation** or data annotations.

---

## Observability

* Centralize logs with **Serilog, Seq, or Application Insights**.
* Use **structured logging** – never log raw objects.
* Expose **health checks** with `/health` endpoint.
* Collect **traces and metrics** with OpenTelemetry.
* Implement **distributed tracing** across services.

---

## Performance

* Use **async/await** everywhere – avoid blocking.
* Benchmark with **BenchmarkDotNet**.
* Optimize serialization with **System.Text.Json** (avoid Newtonsoft.Json unless needed).
* Cache results of expensive operations.
* Pool resources (`HttpClientFactory`, `DbContext pooling`).
* Minimize allocations in high-throughput APIs.

---

## Deployment & Hosting

* Use **Kestrel** with reverse proxy (Nginx/Apache/IIS).
* Enable **gRPC** for internal high-performance communication.
* Deploy as **containers** where possible.
* Apply **Blue-Green/Canary deployments** for zero downtime.
* Enable **health probes** for orchestrators (K8s, App Service).

---

## Testing

* Unit test with **xUnit**.
* Integration test with **TestServer** or **WebApplicationFactory**.
* Contract/API test with **Postman/Newman** or **Playwright**.
* Measure **code coverage** but don’t chase 100%.

---

## Anti-Patterns to Avoid

* Service locator pattern (anti-DI).
* Fat controllers – push logic into services.
* Excessive middleware – keep pipeline lean.
* Not disposing `IDisposable` resources.
* Mixing sync and async code.

---
