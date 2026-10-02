---
name: facade
description: Teach and apply the Facade pattern to provide a simple, task-focused interface over complex subsystems while reducing client coupling to internal classes. Use when users ask about Facade, simplifying third-party frameworks, layered entry points, minimizing integration boilerplate, or comparing Facade with Adapter, Proxy, or Mediator.
---

# Facade

## Purpose
Help the user decide whether Facade is appropriate, then design and implement clear subsystem entry points that isolate client code from complexity and churn.

## When To Use
- User asks to explain Facade pattern.
- Client code is tightly coupled to many subsystem classes and orchestration details.
- Integration requires repeated setup/order/lifecycle code across call sites.
- Team needs stable API boundaries while underlying framework/library evolves.
- Subsystem should be exposed through layer-specific entry points.

## Inputs
- Subsystem classes and current client touchpoints.
- Most common client use cases to prioritize in facade API.
- Lifecycle responsibilities (init, configuration, cleanup, retries, resource handling).
- Error and observability requirements (exceptions, logging, metrics, tracing).
- Constraints: performance, backward compatibility, deployment/migration strategy.

## Workflow
1. Clarify the goal.
   Confirm whether the user needs concept explanation, architecture review, or implementation/refactor.
2. Check applicability.
   Use Facade when the problem is subsystem complexity and coupling, not interface incompatibility.
3. Identify high-value use cases.
   Select the smallest set of client scenarios the facade must support well.
4. Define facade contract.
   Design a concise task-oriented API with clear inputs/outputs and error model.
5. Implement orchestration.
   Move object creation, dependency ordering, and subsystem coordination into facade methods.
6. Isolate subsystem details.
   Keep concrete subsystem types behind facade boundaries; expose domain-oriented results.
7. Migrate clients.
   Redirect client calls from subsystem internals to facade API incrementally.
8. Split when needed.
   If facade grows broad, extract focused facades per bounded context/layer.
9. Validate maintainability.
   Verify client simplification, reduced coupling, and stable behavior under subsystem changes.

## Decision Branches
- If the core issue is incompatible interfaces between two specific classes:
  Prefer Adapter.
- If the goal is access control, lazy loading, or remote indirection with same interface:
  Prefer Proxy.
- If communication between many peer components needs central coordination rules:
  Prefer Mediator.
- If facade starts accumulating unrelated responsibilities:
  Split into additional facades aligned to use-case or layer boundaries.
- If only object creation should be hidden while behavior remains elsewhere:
  Consider Abstract Factory for construction plus optional thin Facade for orchestration.

## Output Contract
When responding, provide:
1. A suitability verdict (Facade or better alternative).
2. Coupling hotspot summary (what client complexity is being removed).
3. Role mapping (Client, Facade, Additional Facade(s), Subsystem components).
4. Incremental migration plan from direct subsystem usage to facade API.
5. Minimal code sketch in the user's language showing one facade method orchestrating multiple subsystem calls.
6. Validation checklist proving reduced client coupling and preserved behavior.

## Quality Checks
- Facade API is minimal, use-case oriented, and free of low-level subsystem jargon.
- Client code depends on facade abstraction, not subsystem internals.
- Facade centralizes lifecycle and ordering concerns consistently.
- Error mapping provides actionable, stable outcomes for clients.
- Subsystem can evolve with limited or no client-side changes.
- Facade size remains bounded; unrelated concerns are split out.

## Common Mistakes To Prevent
- Turning facade into a god object with unrelated features.
- Leaking subsystem types or configuration details through facade API.
- Hiding domain errors without preserving useful context.
- Introducing facade without migrating clients, leaving duplicated integration paths.
- Confusing Facade with Adapter (translation) or Proxy (same-interface control).

## References
- Detailed guide: [REFERENCE.md](./REFERENCE.md)
- Worked examples: [EXAMPLES.md](./EXAMPLES.md)
