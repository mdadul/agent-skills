---
name: singleton
description: Teach and apply the Singleton pattern to ensure a class has exactly one instance with a global access point. Use when users ask about Singleton, single shared instance, thread-safe initialization, lazy vs eager instantiation, or comparing Singleton to dependency injection.
---

# Singleton

## Purpose
Help the user decide whether Singleton is appropriate, then design and implement it correctly — including thread-safety, testability, and common pitfalls.

## When To Use
- User asks to explain the Singleton pattern.
- A shared resource (database connection, config, logger, thread pool) must have exactly one instance.
- Global variables are being used but need protection against arbitrary overwriting.
- Lazy initialization of an expensive resource is required.
- User needs to control the number of instances of a class (Singleton is the single-instance special case).

## Inputs
- The class or resource that should be restricted to one instance.
- Whether the environment is single-threaded or multi-threaded (drives thread-safety strategy).
- Whether initialization must be lazy (on first use) or eager (at startup).
- Language and framework constraints (affects static fields, class loaders, serialization).
- Testability requirements (drives whether pure Singleton or DI-managed singleton is preferred).

## Workflow
1. Clarify the goal.
   Confirm whether the user wants explanation, code review, or implementation/refactor.
2. Check applicability.
   Validate the need: is a single shared instance truly required, or would dependency injection of a single instance be cleaner?
3. Choose initialization strategy.
   Select eager, lazy (null-check), double-checked locking, or initialization-on-demand holder based on thread-safety and startup cost.
4. Implement the core structure.
   Private constructor, private static instance field, public static `getInstance()` / `instance` accessor.
5. Address thread safety.
   Apply the chosen strategy; call out language-specific guarantees (e.g., Go `sync.Once`, Java `volatile`, Kotlin `object`).
6. Address testability.
   If unit testing matters, discuss abstracting behind an interface so tests can inject a substitute.
7. Validate behavior.
   Confirm that multiple `getInstance()` calls return the same reference and that the constructor is unreachable from outside.
8. Explain tradeoffs.
   Call out SRP violation, global-state coupling, and test friction vs convenience.

## Decision Branches
- If the language has a built-in singleton construct (Kotlin `object`, Python module-level instance, Go package-level var with `sync.Once`):
  Prefer the idiomatic language construct over a hand-rolled implementation.
- If testability is a hard requirement:
  Recommend registering the singleton via a DI container as a shared scope rather than using the static getInstance pattern.
- If thread safety is required and initialization is expensive:
  Use double-checked locking with `volatile` (Java/C#) or initialization-on-demand holder idiom.
- If startup cost is negligible:
  Prefer eager initialization — simpler and inherently thread-safe.
- If multiple related singletons are needed:
  Consider whether Abstract Factory or a DI container manages them better than hand-rolled statics.

## Output Contract
When responding, provide:
1. A concise suitability verdict (why Singleton or why not).
2. The chosen implementation strategy and why it fits the constraints.
3. A minimal code sketch in the user's language showing the core structure.
4. Thread-safety notes specific to the user's language/runtime.
5. Testability guidance (interface extraction, DI alternative, or mock strategy).
6. Validation checklist.

## Quality Checks
- Constructor is private (or equivalent); no external `new` calls possible.
- `getInstance()` always returns the same instance reference.
- Thread-safe under concurrent first-call scenarios.
- Instance is not replaced or nulled out after initialization.
- If serializable, `readResolve()` (or equivalent) prevents deserialization creating a second instance.
- Tests can run without depending on shared singleton state leaking between test cases.

## Common Mistakes To Prevent
- Missing `volatile` on the instance field in double-checked locking (Java/C#).
- Assuming module-level initialization is thread-safe without verifying language guarantees.
- Making the singleton hold mutable shared state without synchronization on individual methods.
- Using Singleton where dependency injection of a scoped/shared instance would be testable and cleaner.
- Forgetting that subclassing a Singleton breaks the uniqueness guarantee.
- Not resetting singleton state between unit tests, causing test order dependencies.

## References
- Detailed guide: [REFERENCE.md](./REFERENCE.md)
- Worked examples: [EXAMPLES.md](./EXAMPLES.md)
