# State Reference

## Intent
State allows an object to alter its behavior when its internal state changes. The object will appear to change its class.

## Problem Signal
- A class has methods with large `if`/`switch` blocks all keyed on the same state field.
- Adding a new state requires editing every method in the class.
- State-specific logic is scattered and duplicated.
- Transition rules are implicit, buried in conditionals, and hard to reason about.

## Roles
- **Context**: The object whose behavior varies by state. Holds a reference to the current `State` object. Delegates all state-specific actions to it. Exposes `setState(state)` so state objects (or client) can trigger transitions. Contains no state-branching logic after refactoring.
- **State** (interface): Declares one method per action/event that produces different behavior per state.
- **Concrete State**: Implements the State interface for one specific state. Contains all logic for that state. May hold a backreference to the context to call context services or trigger transitions.
- **Abstract Base State** (optional): Shared behavior or default (no-op) implementations for states that don't handle every event.

## State Machine Mapping
Before coding, draw the state machine:
```
States:  nodes
Events:  edge labels
Transitions:  directed edges from current state to next state
```
Each directed edge becomes a `context.setState(new TargetState())` call inside a Concrete State's method.

## Transition Ownership Options

| Who calls `setState` | When to choose |
|---|---|
| Concrete State | Most common. State knows the next state after an event. Keeps transition logic co-located with state behavior. |
| Context | Use when the context has information states don't have access to (e.g., external conditions). |
| Client | Use for simple state machines where the caller decides what state comes next. |

States should never reference each other's constructors directly in complex machines — consider a state factory or passing state instances via context to reduce coupling.

## Applicability
Use when:
- Object behavior varies significantly across a finite number of named states.
- State-branching conditionals are growing and making the class hard to maintain.
- New states are added regularly and must not break existing code.
- Transition logic needs to be explicit and traceable.

Avoid when:
- Only 2–3 stable states exist with simple branching — an enum + switch is clearer.
- States are identical in behavior and differ only in data — use a single state class parameterized with data.
- The "state" changes rarely and affects only one method — extract that method instead.

## Refactor Recipe
1. List all distinct states by examining the state field values used in conditionals.
2. Draw the state machine diagram.
3. Declare the State interface with one method per state-varying action.
4. Create one Concrete State class per state.
5. Move each conditional branch from context methods into the corresponding Concrete State method.
6. Add a backreference to context in states that need to trigger transitions or call context services.
7. Replace all state conditionals in the context with `this.state.action()` delegation.
8. Add `setState(state)` to the context.
9. Initialize context with the correct starting state in its constructor.
10. Delete the original state field (now implicit in the object type of `this.state`).

## Shared State Optimization
If state objects are stateless (hold no per-context data), they can be shared across context instances as Flyweights — one instance of each state class, reused by all contexts.

```typescript
// Shared singleton states (valid only when state holds no per-context data)
const LOCKED   = new LockedState();
const READY    = new ReadyState();
const PLAYING  = new PlayingState();

// Context transitions
player.setState(LOCKED);
```

## Validation Checklist
- Context has zero `if (state == X)` conditionals after refactoring.
- Each Concrete State class covers exactly one state.
- Adding a new state: new class + wire transitions; zero changes to existing states or context.
- Every transition has a documented owner (`state`, `context`, or `client`).
- Context is initialized with a valid starting state.
- All methods in the State interface are meaningful for at least most states.

## Pros
- Single Responsibility: each state's behavior is isolated in its own class.
- Open/Closed: new states added without touching existing code.
- Eliminates monolithic conditional branches.
- Transition logic is explicit and co-located with state behavior.

## Cons
- Overkill for simple state machines with few states.
- State classes may become tightly coupled if states reference each other's concrete types.
- More classes to manage.

## Relationship Notes
- **State vs Strategy**: Both use composition and delegation. Key difference: in State, concrete states know about transitions and may reference each other; in Strategy, concrete strategies are completely independent and interchangeable. State models a lifecycle; Strategy models interchangeable algorithms.
- **State vs Bridge**: Structurally similar (context holds a reference to an interface). Bridge separates abstraction from implementation as a permanent structural split; State models temporal behavior changes driven by transitions.
- **State + Command**: Commands can trigger state transitions. The command executes an action and the resulting state change is handled by the current state object.
- **State + Singleton / Flyweight**: Stateless state objects (no per-context fields) can be shared instances — one per state class across all contexts.
