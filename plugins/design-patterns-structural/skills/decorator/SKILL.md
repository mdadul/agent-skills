---
name: decorator
description: Teach and apply the Decorator (Wrapper) pattern to attach new behaviors to objects dynamically via composable wrapper layers. Use when users ask about Decorator, Wrapper, adding behavior at runtime, avoiding subclass explosion from optional features, or comparing Decorator to Proxy, Adapter, or Composite.
---

# Decorator

Also known as: Wrapper

## Purpose
Help the user decide whether Decorator is appropriate, then design and apply it to replace inheritance-based behavior extension with composable, stackable wrapper objects.

## When To Use
- User asks to explain the Decorator or Wrapper pattern.
- A class has optional behaviors that are being combined through subclassing, leading to a combinatorial explosion.
- Behaviors need to be added to or removed from individual objects at runtime, not the whole class.
- The class to extend is `final` or otherwise closed to inheritance.
- Multiple independent optional behaviors need to be composed in arbitrary combinations.
- User wants to layer cross-cutting concerns (logging, caching, compression, encryption, auth) over a core component.

## Inputs
- The core component and its interface (the operations clients depend on).
- The set of optional behaviors to attach (each becomes a Concrete Decorator).
- Whether behaviors must be composable in any order, or ordering matters.
- Language constraints (final classes, interface support, decorator language features).
- Whether a base decorator class is worth adding or single-method interfaces make it redundant.

## Workflow
1. Clarify the goal.
   Confirm whether the user wants explanation, code review, or implementation/refactor.
2. Check applicability.
   Confirm there are optional behaviors that vary per object instance, not per class. If all objects always get all behaviors, inheritance may be simpler.
3. Define the Component interface.
   Extract the methods shared by the core component and all decorators into a single interface.
4. Implement the Concrete Component.
   Create the base class with default behavior. Keep it focused — no optional logic here.
5. Create the Base Decorator.
   Implement the Component interface. Hold a `wrappee: Component` reference. Delegate all methods to `wrappee`. This class exists to avoid repeating delegation boilerplate in every Concrete Decorator.
6. Implement Concrete Decorators.
   Each overrides one or more methods, calls `super` / `wrappee` to get the base result, then adds behavior before or after.
7. Compose in the Client.
   Client wraps the Concrete Component in the desired decorators in the needed order. The outermost decorator is what the client interacts with.
8. Validate behavior.
   Confirm each decorator works standalone and in combination. Confirm client code needs no changes when a new decorator is added.
9. Explain tradeoffs.
   Call out stack ordering effects, difficulty removing a mid-stack decorator, and initial wiring verbosity.

## Decision Branches
- If the behavior to add does not change the interface (same methods, extra logic):
  Decorator is a strong fit.
- If the interface itself must change (new methods exposed):
  Decorator can extend the interface, but clients needing the new methods must know the concrete decorator type — evaluate whether this breaks transparency.
- If only one extra behavior is needed permanently for all instances:
  Prefer simple subclassing or composition without the full Decorator scaffolding.
- If controlling access, lifecycle, or identity of an object is the goal:
  Use Proxy instead — structurally similar but different intent.
- If adapting an incompatible interface:
  Use Adapter — Decorator keeps or extends the same interface.
- If passing requests through a chain where any handler may stop the flow:
  Use Chain of Responsibility — decorators must not break the request flow.

## Output Contract
When responding, provide:
1. A concise suitability verdict (why Decorator or why not).
2. Named roles mapped to the user's domain (Component interface, Concrete Component, Base Decorator, Concrete Decorators).
3. Step-by-step migration plan from current class structure to Decorator.
4. A minimal code sketch in the user's language showing wrapping and stacking.
5. Validation checklist confirming transparency and composability.

## Quality Checks
- All decorators and the Concrete Component implement the same Component interface.
- Base Decorator delegates every method to the wrappee — no logic leaks into it.
- Each Concrete Decorator adds exactly one responsibility.
- Client code works identically whether it holds a plain component or a deeply decorated one.
- Adding a new decorator requires zero changes to existing components or other decorators.
- Decorator stack order is documented where ordering produces different results.

## Common Mistakes To Prevent
- Putting business logic in the Base Decorator instead of keeping it as pure delegation.
- Forgetting to delegate methods not being decorated (causing silent behavior drops).
- Making a decorator depend on the concrete type of its wrappee (breaks transparency).
- Confusing Decorator with Proxy: Decorator enriches behavior; Proxy controls access/lifecycle.
- Confusing Decorator with Chain of Responsibility: decorators cannot stop the call chain; CoR handlers can.
- Order-sensitive stacks with no documentation — leads to subtle bugs.

## References
- Detailed guide: [REFERENCE.md](./REFERENCE.md)
- Worked examples: [EXAMPLES.md](./EXAMPLES.md)
