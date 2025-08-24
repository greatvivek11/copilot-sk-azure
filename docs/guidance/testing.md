# Testing Best Practices Guide

## Philosophy

* **Shift-left**: write tests with the code. Prefer **fast, deterministic** tests.
* **Test pyramid**: many unit, some integration, few e2e. Avoid inverted cones.
* **Behavior over implementation**: treat tests as **specs**; assert outcomes, not internals.
* **Integration-first mindset**: prefer thin unit tests + strong integration tests for real boundaries.
* **Small, readable, isolated**: no hidden coupling; fresh state per test.

## Tooling Stack (Recommended)

* **Frontend**: Jest/Vitest + React Testing Library (RTL), Playwright (e2e), Lighthouse CI (perf/a11y), axe-core (a11y).
* **Backend (.NET)**: xUnit, FluentAssertions, Testcontainers for DB/Infra, WireMock.Net/MockHttp for HTTP.
* **API**: Playwright API testing or REST Assured (Java) equivalent; Postman/Newman only for smoke.
* **Contract testing**: Pact (consumer/provider) when teams deploy independently.
* **Mutation testing**: Stryker.NET / StrykerJS to catch weak assertions.

## Unit Tests (Frontend)

* Test **public surface** of components (props/DOM), not internal hooks.
* Use **RTL queries** by role/text/label; avoid test IDs unless necessary.
* Mock network with **MSW (Mock Service Worker)**.
* Keep snapshots scarce and focused.

```tsx
// Button.test.tsx
render(<Button onClick={fn} label="Save" />);
await user.click(screen.getByRole('button', { name: /save/i }));
expect(fn).toHaveBeenCalled();
```

## Unit Tests (Backend, xUnit)

* One **assert** concept per test; use **FluentAssertions** for clarity.
* Isolate pure logic; mock only **external boundaries** (HTTP, clock, DB, FS).
* Prefer **test data builders** over object mothers.

```csharp
[Fact]
public void Money_Add_Should_Sum_Currency() {
    var a = Money.From(10, "USD");
    var b = Money.From(5, "USD");
    (a + b).Amount.Should().Be(15);
}
```

## Integration Tests (.NET Minimal API)

* Use **WebApplicationFactory** to host the real pipeline.
* Use **Testcontainers** to spin real dependencies (SQL, Redis, Kafka).
* Seed DB per test; run tests **in parallel** when isolation is guaranteed.

```csharp
public class ApiFactory : WebApplicationFactory<Program> {
  protected override void ConfigureWebHost(IWebHostBuilder b) =>
    b.ConfigureAppConfiguration((_, cfg) => cfg.AddInMemoryCollection(new Dictionary<string,string?> {
      ["ConnectionStrings:Db"] = TestDb.ConnectionString
    }));
}
```

## E2E Tests (Playwright)

* Keep E2E **thin**: critical journeys only (login, checkout, core flows).
* Make tests **idempotent**: unique test data, clean up on teardown.
* Run on **CI headless** with trace/video only on failure.

```ts
// login.spec.ts
await page.goto('/login');
await page.getByLabel('Email').fill(user.email);
await page.getByLabel('Password').fill(user.password);
await page.getByRole('button', { name: 'Sign in' }).click();
await expect(page).toHaveURL('/dashboard');
```

## API Testing

* Prefer **Playwright request** or **Supertest** against a real server.
* Validate **status, headers, body, schema**; use Zod/JSON Schema for response contracts.
* Use **contract tests (Pact)** when teams deploy independently.

```ts
const res = await request.get('/api/orders/42');
expect(res.status()).toBe(200);
expect(schema.parse(await res.json())).toBeTruthy();
```

## Databases & EF Core

* Avoid EF InMemory provider for behavior; use **SQLite In-Memory** or **real DB via Testcontainers**.
* Verify generated SQL with `.ToQueryString()` in targeted tests.
* Reset DB state per test (transaction rollback or recreate schema).

## Test Data Management

* Use **builders + factories** (e.g., Bogus for realistic data).
* Avoid shared fixtures that leak state; prefer **per-test** setup.
* Record/replay HTTP with **WireMock** for consistent responses.

```csharp
class UserBuilder { public User Build() => new("u"+Guid.NewGuid(), "user@example.com"); }
```

## Coverage & Quality Gates

* Track **branch coverage**; set realistic thresholds (e.g., 70–85%).
* Use **mutation testing** to assess assertion strength.
* Coverage is a **signal, not a goal**; target risky modules first.

## Non-Functional Testing

* **Performance**: k6 or Artillery for API throughput/p95; Lighthouse CI for web vitals.
* **Security**: OWASP ZAP baseline scans in CI; dependency audits (npm audit/dotnet list package --vuln).
* **Accessibility**: axe-core + manual keyboard checks.
* **Resilience**: chaos tests on retry/backoff logic (fault injection).

## CI/CD Integration

* Run **lint → unit → integration → e2e (smoke)** in pipelines.
* Parallelize test suites; cache dependencies.
* Publish **JUnit/Trx** results and HTML reports; fail fast on flakies.
* Tag/flaky tests and auto-retry **once** only; fix root causes ASAP.

```yaml
# github actions sketch
jobs:
  test:
    strategy: { matrix: { node: [18, 20] } }
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm ci && npm run lint && npm run test:unit -- --ci
      - run: npm run test:e2e -- --reporter=junit
```

## Best Practices (Checklist)

* [ ] Tests are **fast, isolated, deterministic**.
* [ ] Unit tests cover **pure logic**, integration covers **boundaries**.
* [ ] E2E covers **happy paths + critical regressions** only.
* [ ] MSW/WireMock for external dependencies.
* [ ] Playwright traces on failure; video/screenshots optional.
* [ ] DB tests run on **real engines** via Testcontainers.
* [ ] Coverage + mutation testing in CI as signals.
* [ ] Perf/a11y/security checks in CI, not just prod.

## Anti-Patterns

* Over-mocking internals; brittle tests that mirror implementation.
* Global shared state/fixtures causing test order dependence.
* Excessive E2E coverage that slows feedback drastically.
* Ignoring **time, randomness, and clock** determinism (use fake timers/clock).
* Using prod secrets or real 3rd-party APIs in CI.
