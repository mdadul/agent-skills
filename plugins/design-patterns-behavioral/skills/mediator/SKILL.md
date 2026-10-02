---
name: mediator
description: Teach and apply the Mediator pattern to centralize collaboration rules among tightly coupled components and eliminate direct component-to-component dependencies. Use when users ask about Mediator, Intermediary, Controller, UI dialog coordination, reusable components, or comparing Mediator with Observer, Facade, or Chain of Responsibility.
---

# Mediator

Also known as: Intermediary, Controller

## Purpose
Help the user decide whether Mediator is appropriate, then design and implement centralized coordination objects that reduce dependency chaos and improve component reuse.

## When To Use
- User asks to explain Mediator pattern.
- Components are tightly coupled through many direct references.
- Reusing a component is difficult because it depends on many peers.
- UI/dialog workflows require cross-component reactions and state synchronization.
- Team needs to change interaction rules without editing every component class.

## Inputs
- Set of interacting components and current communication graph.
- Events/notifications emitted by components.
- Coordination rules currently spread across components.
- Lifecycle ownership decisions (who creates and wires components).
- Constraints: performance, testability, and required observability.

## Workflow
1. Clarify the goal.
   Confirm whether the user needs concept explanation, architecture review, or implementation/refactor.
2. Check applicability.
   Use Mediator when component interdependencies are the core maintainability problem.
3. Define mediator contract.
   Introduce a mediator interface with notification protocol (sender + event + context).
4. Implement concrete mediator.
   Move inter-component coordination logic into mediator class.
5. Wire components to mediator.
   Replace direct peer calls with mediator notifications.
6. Normalize events.
   Define consistent event names/payloads to avoid ad hoc coupling through strings.
7. Isolate responsibilities.
   Keep components focused on local behavior; keep cross-component rules in mediator.
8. Split oversized mediators.
   Extract additional mediators by feature slice or bounded context when coordination grows.
9. Validate decoupling.
   Verify components are reusable with alternate mediators and no longer depend on peer concrete types.

## Decision Branches
- If communication is one-to-many event broadcast with dynamic subscriptions:
  Prefer Observer.
- If goal is simplifying external use of a subsystem rather than coordinating peers:
  Prefer Facade.
- If request should pass through ordered handlers until processed:
  Prefer Chain of Responsibility.
- If mediator starts to become a god object:
  Split by interaction domain or compose mediator with smaller policy objects.
- If component interactions are simple and stable:
  Prefer direct collaboration for lower complexity.

## Output Contract
When responding, provide:
1. A suitability verdict (Mediator or better alternative).
2. Dependency-chaos summary (what coupling is being removed).
3. Role mapping (Mediator interface, Concrete Mediator, Components, Client/wiring).
4. Incremental migration plan from direct peer calls to mediator notifications.
5. Minimal code sketch in the user's language showing two components interacting through mediator.
6. Validation checklist proving decoupling and behavior parity.

## Quality Checks
- Components communicate through mediator interface only.
- Components no longer reference peer concrete classes.
- Mediator coordination logic is explicit, cohesive, and testable.
- Event protocol is stable and documented.
- New interaction rules can be added in mediator without changing components.
- Mediator scope remains bounded; split when unrelated rules accumulate.

## Common Mistakes To Prevent
- Moving all business logic into mediator and creating a god object.
- Using unstructured event payloads that reintroduce hidden coupling.
- Keeping backdoor direct component calls after mediator introduction.
- Treating mediator as global singleton without bounded context rationale.
- Confusing Mediator with Facade or Observer intent.

## References
- Detailed guide: [REFERENCE.md](./REFERENCE.md)
- Worked examples: [EXAMPLES.md](./EXAMPLES.md)
