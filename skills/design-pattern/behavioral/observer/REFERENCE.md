# Observer Reference

## Intent
Observer defines a subscription mechanism where publishers notify multiple subscribers about events without depending on their concrete classes.

## Problem Signal
- Consumers repeatedly poll for changes.
- Publisher has hard-coded knowledge of all listeners.
- Adding new reactions requires editing core publisher logic.
- Runtime listener lifecycles are difficult to manage.

## Solution Shape
1. Define subscriber interface/callback contract.
2. Add subscription infrastructure to publisher.
3. Emit events when meaningful state changes occur.
4. Let subscribers register/unregister dynamically.
5. Keep publisher unaware of concrete subscriber types.

## Roles
- Publisher/Subject: Emits events and manages subscriber list.
- Subscriber/Observer: Receives update notifications.
- Concrete Subscriber: Performs specific side effects/actions.
- Client/Wiring: Creates and registers subscribers at runtime.

## Applicability
Use when:
- One state source must notify many independent dependents.
- Subscriber population is dynamic or plugin-like.
- Runtime extensibility without publisher edits is required.

Avoid when:
- Subscriber set is tiny and static with simple direct calls.
- Strong orchestration logic across peers is needed (Mediator fits better).

## Event Contract Notes
- Prefer explicit event types and versioned payload schemas.
- Keep payload minimal but sufficient for subscriber work.
- Decide push model (full payload) vs pull model (publisher reference/context).
- Document ordering, retries, and failure isolation policy.

## Refactor Recipe
1. Identify state changes currently polled or manually fanned out.
2. Define event names and payload models.
3. Add `subscribe`/`unsubscribe`/`notify` to publisher.
4. Extract each reaction into a subscriber implementation.
5. Replace direct callbacks with subscription wiring.
6. Add cleanup hooks and tests for lifecycle correctness.

## Validation Checklist
- Subscribers can be added/removed without publisher code changes.
- Unsubscribed listeners receive no further events.
- Subscriber failures are isolated or handled per policy.
- Event payloads remain backward-compatible where required.
- Notification fan-out performs within acceptable latency limits.

## Pros
- Decouples publisher from concrete listeners.
- Supports runtime extensibility and dynamic wiring.
- Encourages event-driven architecture boundaries.

## Cons
- Notification order may be nondeterministic unless enforced.
- Risk of leaks or event storms if lifecycle/backpressure is unmanaged.

## Relationship Notes
- Mediator centralizes interaction logic between peers; Observer broadcasts to subscribers.
- Command encapsulates executable operations; Observer distributes state-change notifications.
- CoR routes one request through handlers; Observer fans one event out to many listeners.
