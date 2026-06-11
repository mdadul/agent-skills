# Chain of Responsibility Reference

## Intent
Chain of Responsibility lets you pass a request along a chain of handlers. Each handler decides whether to process the request or forward it to the next handler, decoupling the sender from the receiver.

## Problem Signal
- A growing sequence of checks or processing steps has become a monolithic, tangled block.
- Checks need to be reordered, added, or removed without touching unrelated code.
- The same checks are duplicated across different call sites because they cannot be independently composed.
- The sender should not know which object will ultimately handle the request.

## Variants

### Stop-on-First (Classic)
Each handler either handles the request and stops, or passes it on without handling.
- One handler processes the request (or none).
- Used for: event bubbling, support escalation, permission lookups, command dispatch.

### Pass-Through (Pipeline)
Every handler in the chain processes the request; none stops propagation.
- All handlers run in order; each may augment, validate, or transform the request.
- Used for: HTTP middleware, logging pipelines, data processing pipelines, sequential validation.

## Roles
- **Handler** (interface): Declares `handle(request)` and optionally `setNext(handler): Handler`.
- **Base Handler** (abstract, optional): Implements Handler. Holds `next: Handler`. Default `handle()` forwards to `next` if present. Concrete handlers call `super.handle(request)` to propagate.
- **Concrete Handler**: Checks whether it should process the request; processes it or forwards it (stop-on-first) or processes it and always forwards (pass-through).
- **Client**: Assembles the chain (via `setNext` or constructor injection) and sends requests to any handler in the chain — not necessarily the first one.

## Applicability
Use when:
- Multiple objects may handle a request and the handler is not known at design time.
- Handlers and their order must be configurable or changeable at runtime.
- Request processing logic must be decoupled, independently testable, and reusable.

Avoid when:
- The handling sequence is fixed and simple — a direct call chain is clearer.
- Every request must be guaranteed to be handled — CoR can silently drop requests (add explicit end-of-chain handling if this matters).
- A single handler always processes all requests (no need for a chain of one).

## Refactor Recipe
1. Identify the monolithic sequential-check block.
2. Declare a Handler interface with `handle(request)`.
3. Create a Base Handler: `next` field, default `handle()` that forwards.
4. Extract each check into its own Concrete Handler class.
5. Move per-handler logic into each `handle()` method.
6. Assemble the chain in a factory, configurator, or client.
7. Replace the original monolithic code with a call to the chain's entry point.
8. Add explicit end-of-chain behavior (log, throw, return default) if unhandled requests must not be silently dropped.

## Validation Checklist
- Each handler class has one responsibility.
- No handler holds a reference to a concrete handler type — only the Handler interface.
- Adding a new handler requires zero changes to existing handlers.
- Reordering requires changes only in the assembly code.
- Unhandled requests produce a predictable, intentional outcome.
- Chain has a defined termination (last handler, null check, or default end handler).

## Pros
- Single Responsibility: each handler handles one concern.
- Open/Closed: new handlers added without changing existing ones.
- Runtime chain composition and reordering.
- Decouples request sender from receiver.

## Cons
- Requests may go unhandled (especially in stop-on-first variant with no default).
- Long chains are hard to trace and debug.
- No guarantee of processing in pass-through variant if a handler throws and doesn't forward.

## Relationship Notes
- **CoR vs Decorator**: Both use recursive composition. Decorator cannot stop propagation — it must always forward. CoR can stop at any point. Decorator enriches behavior; CoR routes/handles requests.
- **CoR vs Command**: Handlers can be implemented as Commands. Alternatively, the request itself can be a Command object passed along the chain.
- **CoR vs Mediator**: CoR passes requests linearly along a chain; Mediator centralizes all communication between many objects through one hub.
- **CoR vs Observer**: CoR passes to one (or all) handlers sequentially; Observer fans out to all subscribers simultaneously.
- **CoR + Composite**: A Composite tree naturally forms a CoR chain — a leaf forwards to its parent container, which forwards to its parent, up to the root. GUI event bubbling is the canonical example.
- **CoR, Command, Mediator, Observer**: All address connecting request senders and receivers, but differ in topology (linear vs. star vs. broadcast) and intent.
