---
name: chain-of-responsibility
description: Teach and apply the Chain of Responsibility (CoR) pattern to pass requests along a dynamic chain of handlers where each can process or forward the request. Use when users ask about CoR, Chain of Command, middleware pipelines, sequential validation, request bubbling, or comparing CoR to Decorator, Command, or Mediator.
---

# Chain of Responsibility

Also known as: CoR, Chain of Command

## Purpose
Help the user decide whether Chain of Responsibility is appropriate, then design and apply it to decouple request senders from receivers and replace tangled sequential-check code with a composable, ordered chain of focused handler objects.

## When To Use
- User asks to explain the Chain of Responsibility or Chain of Command pattern.
- Code has a growing sequence of checks/validations that have become monolithic and hard to reorder or reuse.
- The set of handlers or their order may change at runtime.
- Multiple handlers may need to process the same request, or exactly one handler should process it — depending on the variant.
- Request handling logic must be reusable across different pipelines without duplication.
- User needs to implement middleware, event bubbling, support escalation, or access-control pipelines.

## Inputs
- The request type and its data (what is being passed along the chain).
- The set of handlers needed and their logical order.
- Which CoR variant applies: pass-through (all handlers run) or stop-on-first (first capable handler takes it).
- Whether chain composition must be dynamic (runtime) or static (compile time).
- Language and framework constraints (middleware infrastructure, event systems, etc.).

## Workflow
1. Clarify the goal.
   Confirm whether the user wants explanation, code review, or implementation/refactor. Identify the variant: pass-through pipeline vs. stop-on-first handler.
2. Check applicability.
   Confirm requests need sequential processing through independent handlers. If all logic always runs in a fixed order with no interleaving, a simple sequential call chain may suffice.
3. Define the Handler interface.
   Declare a `handle(request)` method. Optionally declare `setNext(handler)` if chaining will be explicit.
4. Create the Base Handler (optional but recommended).
   Implement the Handler interface. Hold a `next: Handler` reference. Provide a default `handle()` that forwards to `next` if set. Concrete handlers call `super.handle()` to propagate.
5. Implement Concrete Handlers.
   Each handler checks whether it should process the request. In stop-on-first: process and return, or forward. In pass-through: process (possibly augmenting the request), then always forward.
6. Compose the chain.
   Client (or a factory/configurator) links handlers by calling `setNext` or constructor injection. Chain can start from any handler — not necessarily the first.
7. Trigger the chain.
   Client sends the request to the entry point. Handle unhandled requests explicitly (log, throw, return a default).
8. Validate behavior.
   Confirm adding a new handler requires no changes to existing handlers or clients. Confirm reordering works by changing only the assembly code.
9. Explain tradeoffs.
   Call out the risk of unhandled requests, debugging difficulty in long chains, and the difference from Decorator.

## Decision Branches
- If every handler must always run (pipeline, not early-exit):
  Use pass-through variant — every handler processes then forwards unconditionally.
- If the first capable handler should claim the request and stop propagation:
  Use stop-on-first variant — handler either processes or forwards, never both.
- If behaviors must wrap a call and cannot stop it:
  Use Decorator instead — Decorator cannot stop the call chain; CoR can.
- If a single handler orchestrates communication between many objects:
  Use Mediator instead.
- If the sender needs a record of the operation for undo/redo:
  Combine with Command — the request itself becomes a Command object.
- If the chain maps to a Composite tree structure:
  CoR naturally emerges from the tree — each node forwards to its parent.

## Output Contract
When responding, provide:
1. A suitability verdict (why CoR or why not) and which variant applies.
2. Named roles mapped to the user's domain.
3. Step-by-step migration plan from current monolithic check code to handler chain.
4. A minimal code sketch in the user's language showing at least two handlers and chain assembly.
5. Guidance on handling unhandled requests.
6. Validation checklist.

## Quality Checks
- Each handler has exactly one responsibility.
- Handlers do not reference each other's concrete types — only the Handler interface.
- Adding a new handler requires zero changes to existing handlers or clients.
- Reordering the chain requires changes only in the assembly / configuration code.
- Unhandled requests are handled explicitly (not silently dropped unless by design).
- In pass-through variant: every handler always forwards; no handler stops the chain.
- In stop-on-first variant: a handler never both processes and forwards the same request.

## Common Mistakes To Prevent
- Handlers calling each other directly (bypassing the chain interface) — breaks decoupling.
- Silent request drops when no handler matches and no default end-of-chain behavior exists.
- Mixing pass-through and stop-on-first logic in the same chain without clear documentation.
- Creating a handler that both processes the request AND passes it on in a stop-on-first chain.
- Confusing CoR with Decorator: CoR can stop propagation; Decorator cannot.
- Infinitely long or circular chains with no termination condition.

## References
- Detailed guide: [REFERENCE.md](./REFERENCE.md)
- Worked examples: [EXAMPLES.md](./EXAMPLES.md)
