---
name: observer
description: Teach and apply the Observer pattern to implement event-driven one-to-many notifications with dynamic subscription and loose coupling between publishers and subscribers. Use when users ask about Observer, Event-Subscriber, Listener, pub/sub style updates, runtime subscriptions, or comparing Observer with Mediator, Command, or Chain of Responsibility.
---

# Observer

Also known as: Event-Subscriber, Listener

## Purpose
Help the user decide whether Observer is appropriate, then design and implement subscription-based notification flows that decouple event sources from interested listeners.

## When To Use
- User asks to explain Observer pattern.
- State changes in one object should notify many dependent objects.
- Subscriber set is unknown at design time or changes at runtime.
- Polling is wasteful and event-driven notification is preferred.
- Feature teams need to add reactions without editing publisher logic.

## Inputs
- Publisher(s) and event types to expose.
- Subscriber responsibilities and filtering needs.
- Event payload shape (minimal data vs full context object).
- Delivery requirements (sync/async, ordering, retries, backpressure).
- Lifecycle constraints (subscribe/unsubscribe timing, memory-leak prevention).

## Workflow
1. Clarify the goal.
   Confirm whether the user needs conceptual guidance, architecture review, or implementation/refactor.
2. Check applicability.
   Use Observer when one-to-many notifications and dynamic listeners are core needs.
3. Define contracts.
   Create publisher subscription API (`subscribe`, `unsubscribe`, `notify`) and subscriber interface (`update` or handler callback).
4. Model event schema.
   Specify event names/payloads and versioning strategy to prevent fragile coupling.
5. Implement publisher mechanism.
   Store subscriber registrations and trigger notifications on meaningful state transitions.
6. Implement subscribers.
   Keep subscriber behavior focused and resilient to missing/extra event data.
7. Wire subscriptions at runtime.
   Register subscribers where ownership/lifecycle is clear; ensure cleanup on teardown.
8. Add delivery guarantees.
   Decide ordering, isolation of subscriber failures, and sync vs async dispatch policy.
9. Validate event flow.
   Verify notifications reach intended subscribers, unsubscribed listeners stop receiving events, and publisher remains decoupled.

## Decision Branches
- If coordination among peers needs centralized interaction rules:
  Prefer Mediator.
- If requests should be encapsulated as executable objects with history/undo:
  Prefer Command.
- If a request should flow through ordered handlers until handled:
  Prefer Chain of Responsibility.
- If subscribers are always fixed and static:
  Consider direct collaboration for simplicity.
- If publisher begins to encode subscriber-specific branching:
  Extract event-specific subscribers or intermediate dispatchers.

## Output Contract
When responding, provide:
1. A suitability verdict (Observer or better alternative).
2. Event model summary (publisher, event types, subscriber roles).
3. Role mapping (Publisher, Subscriber Interface, Concrete Subscribers, Client wiring).
4. Incremental migration plan from polling/direct callbacks to subscriptions.
5. Minimal code sketch in the user's language showing subscribe/unsubscribe/notify and one subscriber.
6. Validation checklist proving decoupled and correct notification behavior.

## Quality Checks
- Publisher depends only on subscriber interface/callback signature.
- Subscriber registration lifecycle is explicit and leak-safe.
- Event payload contracts are documented and stable.
- Subscriber failures do not silently break publisher flow unless specified.
- New subscriber types can be added without changing publisher internals.
- Ordering and delivery semantics are explicit and tested.

## Common Mistakes To Prevent
- Publisher containing subscriber-specific logic branches.
- Forgetting unsubscribe/cleanup and causing memory leaks.
- Overloading one event type with ambiguous payloads.
- Unbounded synchronous fan-out causing latency spikes.
- Confusing Observer (broadcast subscriptions) with Mediator (centralized coordination).

## References
- Detailed guide: [REFERENCE.md](./REFERENCE.md)
- Worked examples: [EXAMPLES.md](./EXAMPLES.md)
