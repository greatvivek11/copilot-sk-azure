# JavaScript + TypeScript Best Practices Guide

## Philosophy

* Prefer **TypeScript over JavaScript** for type safety and maintainability.
* Write **modular, functional, immutable-first code**.
* Embrace **simplicity and readability** over clever tricks.
* Treat types as **part of design**, not just implementation detail.
* Automate quality: linters, formatters, tests, and CI/CD.

## Language Practices

* Use `const` by default; `let` only when reassigning; avoid `var`.
* Prefer **arrow functions** for short callbacks, named functions for public APIs.
* Use **strict null checks** and always enable `strict` mode in TypeScript.
* Avoid implicit `any`; use `unknown` when type is truly unknown.
* Use **type aliases** for primitives instead of raw strings/numbers.

```ts
// Bad
function greet(name) { return "Hi " + name; }

// Good
function greet(name: string): string { return `Hi ${name}`; }
```

## Types & Interfaces

* Prefer **interfaces** for object shapes, **types** for unions and primitives.
* Use **readonly** and `ReadonlyArray` for immutability.
* Prefer **discriminated unions** over enums when modeling variants.
* Use **string literal types** instead of magic strings.
* Use **utility types** (`Partial<T>`, `Pick<T>`, `Omit<T>`) to avoid repetition.

```ts
type Status = "draft" | "published" | "archived";

interface Post {
  readonly id: string;
  title: string;
  status: Status;
}
```

## Error Handling

* Always handle **async errors** with `try/catch` or `Result<T, E>` patterns.
* Prefer **custom error classes** over raw `Error`.
* Don’t swallow errors silently.
* In TS, use `never` in exhaustive checks to enforce handling.

```ts
function assertNever(x: never): never {
  throw new Error("Unexpected object: " + x);
}
```

## Async & Concurrency

* Use **async/await** instead of raw Promises.
* Avoid `Promise.all` with mixed failure handling — prefer `Promise.allSettled`.
* Use **AbortController** for cancellable tasks.
* In Node.js, prefer streaming APIs for large data.
* Never block the event loop with heavy sync logic.

## Collections & Data

* Prefer **map/filter/reduce** over manual loops for readability.
* Use `Map`/`Set` over plain objects for keyed collections.
* Deep clone with structuredClone (browser) or libraries, avoid JSON hacks.
* Favor **immutable updates** with spread/rest operators.

```ts
const updated = { ...user, isActive: true };
```

## Patterns & Practices

* Use **Result\<T, E>** or `Either` monads for error-prone logic.
* Embrace **function composition** with small, pure functions.
* Use **static helper classes** sparingly; prefer modules.
* Split large files using **barrel exports** (`index.ts`).
* Use **extension functions** via module augmentation carefully.

## Tooling & Quality

* Enforce formatting with **Prettier**.
* Lint with **ESLint** (with `typescript-eslint`).
* Use **tsc --noEmit** in CI for type checks.
* Add **JSDoc/TSDoc** for public APIs.
* Enable **strictest TS config** (`noImplicitAny`, `strictNullChecks`, `exactOptionalPropertyTypes`).

## Testing

* Unit tests: **Jest / Vitest**.
* Integration: **Playwright** or **Cypress**.
* Use **ts-jest** or built-in `ts-node` for type-safe tests.
* Mock only at boundaries (network, DB, FS).
* Measure coverage but avoid chasing 100%.

```ts
test("adds numbers", () => {
  expect(1 + 2).toBe(3);
});
```

## Anti-Patterns

* Avoid `any` — it removes type safety.
* Avoid deep inheritance hierarchies; use composition.
* Don’t overuse enums; prefer string literals.
* Avoid default exports in libraries (named exports scale better).
* Avoid mixing JS and TS in long-term codebases.

## References

* [TypeScript Handbook](https://www.typescriptlang.org/docs/)
* [ESLint + typescript-eslint](https://typescript-eslint.io/)
* [Airbnb JS Style Guide](https://github.com/airbnb/javascript)
* [Microsoft TS Guidelines](https://github.com/microsoft/TypeScript/wiki/Coding-guidelines)
