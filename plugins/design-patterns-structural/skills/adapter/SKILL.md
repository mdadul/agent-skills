---
name: adapter
description: Teach and apply the Adapter pattern to let incompatible interfaces collaborate by translating calls, data formats, or call order through a wrapper. Use when users ask about Adapter or Wrapper, integrating legacy or third-party services, format conversion boundaries, or comparing Adapter with Decorator, Facade, Proxy, or Bridge.
---

# Adapter

Also known as: Wrapper

## Purpose
Help the user decide whether Adapter is appropriate, then design and implement safe interface translation layers between client code and incompatible services.

## When To Use
- User asks to explain Adapter or Wrapper pattern.
- Client needs to consume a legacy or third-party API with incompatible method signatures or data formats.
- Existing service cannot be changed safely (external dependency, high coupling, source unavailable).
- Integration requires protocol conversion (XML to JSON, units, field mapping, call ordering).
- Multiple incompatible providers should be normalized behind one client-facing contract.

## Inputs
- Target client interface expected by business logic.
- Service interface provided by legacy/third-party system.
- Data and behavior mismatches (types, units, structure, error model, sync vs async).
- Operational constraints (performance overhead, logging, retries, idempotency, error policy).
- Language/runtime constraints (inheritance support, composition preference, DI/testing setup).

## Workflow
1. Clarify the goal.
   Confirm whether the user needs concept explanation, architecture review, or implementation/refactor.
2. Check applicability.
   Use Adapter when the primary need is interface compatibility, not feature extension.
3. Define boundary contracts.
   Document client-facing interface and service-facing interface, including data/error semantics.
4. Choose adapter style.
   Prefer object adapter (composition). Use class adapter only when language and hierarchy constraints justify it.
5. Scaffold adapter API.
   Implement the client interface in adapter class with a wrapped service dependency.
6. Translate calls and data.
   Map arguments, convert payload formats, normalize return values, and bridge exception/error models.
7. Integrate behind abstractions.
   Wire clients to depend on client interface, not concrete adapter/service implementations.
8. Expand safely.
   Add new adapters for additional services/providers without changing client business logic.
9. Validate behavior.
   Verify translation correctness, error propagation, and compatibility under representative scenarios.

## Decision Branches
- If interface is already compatible and only extra behavior is needed:
  Prefer Decorator.
- If goal is simplified entry point for a subsystem (not protocol translation):
  Prefer Facade.
- If interface must remain unchanged while controlling access/lifecycle:
  Prefer Proxy.
- If abstraction and implementation should evolve independently from day one:
  Prefer Bridge.
- If multiple existing clients depend on current service interface and service can be modified safely:
  Consider direct service redesign over adding adapter layer.

## Output Contract
When responding, provide:
1. A suitability verdict (Adapter or a better alternative).
2. Mismatch matrix (interface, data, error, lifecycle differences).
3. Role mapping (Client, Client Interface, Service, Adapter).
4. Incremental migration plan from direct service usage to adapter boundary.
5. Minimal code sketch in the user's language showing translation logic.
6. Validation checklist proving compatibility and non-regression.

## Quality Checks
- Adapter fully implements the client-facing contract.
- Adapter delegates business work to service and only handles translation/boundary concerns.
- Data conversions are explicit, deterministic, and tested.
- Error mapping preserves actionable context for clients.
- Client code references abstractions, not concrete service classes.
- Adding a new service variant requires a new adapter, not client rewrites.

## Common Mistakes To Prevent
- Putting business logic inside adapter instead of pure translation.
- Leaking service-specific types or exceptions through client interface.
- Performing partial conversion that leaves mixed-domain payloads.
- Tight-coupling client code to concrete adapter implementations.
- Overusing Adapter where simpler direct changes would reduce complexity.

## References
- Detailed guide: [REFERENCE.md](./REFERENCE.md)
- Worked examples: [EXAMPLES.md](./EXAMPLES.md)
