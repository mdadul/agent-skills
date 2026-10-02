# Singleton Reference

## Intent
Singleton ensures a class has only one instance and provides a global access point to it.

## Problem Signal
- Shared resource (DB connection, config, cache, logger) is being instantiated multiple times inadvertently.
- Global variables are used to share a single object but can be overwritten anywhere.
- Construction of an expensive object should happen at most once, and that constraint is not enforced.

## Solution Shape
1. Declare a private static field to hold the sole instance.
2. Make the constructor private so no external code can call `new`.
3. Provide a public static method (`getInstance()`) that creates the instance on first call and returns the cached instance on all subsequent calls.

## Roles
- **Singleton class**: Owns the private static instance, the private constructor, and the `getInstance()` accessor. Also contains the business logic executed on the single instance.
- **Client**: Always obtains the instance via `getInstance()`; never calls the constructor directly.

## Initialization Strategies

| Strategy | Thread-safe | Lazy | Notes |
|---|---|---|---|
| Eager (static field init) | Yes | No | Simplest; safe if startup cost is low |
| Lazy with synchronized method | Yes | Yes | Simple but every call pays lock cost |
| Double-checked locking | Yes | Yes | Requires `volatile` (Java/C#); fast after init |
| Initialization-on-demand holder | Yes | Yes | Java idiom; relies on class loader guarantee |
| Language construct (`object`, `sync.Once`, module var) | Yes | Varies | Prefer over hand-rolled when available |

## Applicability
Use when:
- Exactly one instance of a class must exist across the entire program.
- That instance must be accessible from many places without passing it explicitly.
- Lazy initialization of an expensive resource is needed.

Avoid when:
- Testability matters and the singleton holds mutable state (tests become order-dependent).
- A DI container is available — register the type as a singleton scope instead.
- The "single instance" is only a convention, not a hard system constraint.

## Refactor Recipe
1. Identify the class whose instances should be restricted.
2. Add a private static field: `private static instance: MyClass`.
3. Make the constructor private.
4. Add `public static getInstance(): MyClass` with chosen initialization strategy.
5. Replace all `new MyClass()` call sites with `MyClass.getInstance()`.
6. If testability is needed, extract an interface and provide an injection seam.

## Validation Checklist
- `MyClass.getInstance() === MyClass.getInstance()` (same reference every call).
- `new MyClass()` is a compile-time or runtime error from outside the class.
- Concurrent first-call scenario produces only one instance.
- Singleton state does not leak between test cases.

## Pros
- Guarantees single instance across the program.
- Controlled, lazy initialization of expensive resources.
- Global access without passing the object through every call chain.

## Cons
- Violates Single Responsibility Principle (manages own lifecycle AND does business logic).
- Acts as hidden global state; components become implicitly coupled.
- Hard to unit test: private constructor blocks mocking; static method cannot be overridden.
- Requires explicit thread-safety handling in multi-threaded environments.
- Serialization / reflection / multiple class loaders can break the uniqueness guarantee.

## Relationship Notes
- **Facade**: A Facade is often made a Singleton since one facade object is sufficient.
- **Flyweight vs Singleton**: Flyweight allows many instances with shared intrinsic state; Singleton allows only one. Flyweight objects are immutable; Singleton may be mutable.
- **Abstract Factory / Builder / Prototype**: Any of these can be implemented as a Singleton.
- **DI container alternative**: Most modern DI frameworks manage singleton scope explicitly — prefer that over hand-rolled statics when a container is present.
