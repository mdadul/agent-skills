# Mediator Reference

## Intent
Mediator centralizes communication rules between components, replacing direct peer dependencies with indirect coordination through a mediator object.

## Problem Signal
- Components contain many references to each other.
- Small interaction changes force edits across many classes.
- Components are hard to test or reuse due to peer coupling.
- UI/dialog coordination logic is duplicated in controls.

## Solution Shape
1. Define mediator interface for component notifications.
2. Move inter-component decision logic into concrete mediator.
3. Make components notify mediator instead of calling peers.
4. Keep components reusable by depending only on mediator interface.
5. Optionally let mediator own wiring/lifecycle where practical.

## Roles
- Mediator Interface: Contract used by components to report events.
- Concrete Mediator: Encapsulates orchestration and routing rules.
- Component: Local behavior + notification to mediator.
- Client/Wiring: Creates mediator and components, links them.

## Applicability
Use when:
- Inter-component dependency graph has become chaotic.
- Reusability and isolated testing are blocked by peer coupling.
- You need to evolve interaction policies frequently.

Avoid when:
- Interactions are simple and unlikely to change.
- Broadcast event model with dynamic subscriptions is primary (Observer may be better).

## Refactor Recipe
1. Map current component interaction graph.
2. Define notification protocol (event + sender + context).
3. Create concrete mediator and move cross-component rules.
4. Replace direct component calls with `mediator.notify(...)`.
5. Remove stale peer dependencies and simplify components.
6. Add tests for mediator rule scenarios and component isolation.

## Validation Checklist
- Component classes compile and run without peer references.
- Interaction changes are localized to mediator.
- Components can be reused with a different mediator implementation.
- Mediator behavior is covered by focused tests.
- No hidden backchannels bypassing mediator contract.

## Pros
- Reduces coupling between components.
- Improves maintainability of interaction rules.
- Enables component reuse across contexts with alternate mediators.

## Cons
- Risk of mediator becoming too large/god-like.
- Adds an extra indirection layer and coordination complexity.

## Relationship Notes
- Observer distributes notifications to subscribers; Mediator centralizes bilateral/multilateral coordination rules.
- Facade simplifies external access to a subsystem; components inside subsystem can still talk directly.
- CoR routes a request through ordered handlers; Mediator chooses collaborators based on context.
- Command can be used within mediator actions when interactions should be queued/undoable.
