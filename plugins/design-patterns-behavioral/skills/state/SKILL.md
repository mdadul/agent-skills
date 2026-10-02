---
name: state
description: Teach and apply the State pattern to replace state-based conditional branches with polymorphic state objects. Use when users ask about State, finite-state machines, eliminating state conditionals, behavior that changes per state, or comparing State to Strategy.
---

# State

## Purpose
Help the user decide whether State is appropriate, then design and apply it to extract state-specific behavior out of bloated conditionals into a set of focused, independently maintainable state classes.

## When To Use
- User asks to explain the State pattern.
- A class has methods full of `if`/`switch` blocks that branch on the same state field.
- Behavior changes significantly per state and the number of states is growing.
- State-specific code is duplicated across multiple methods in the same class.
- New states need to be added frequently without modifying existing ones.
- State transitions need to be clearly documented and centrally defined.

## Inputs
- The context class and the state field(s) driving conditional branches.
- The set of states (finite, named, enumerable).
- The set of actions/events that trigger state transitions.
- Who is responsible for initiating transitions: the context, the state objects, or the client.
- Language constraints (nested classes, sealed/final states, enum-based state machines).

## Workflow
1. Clarify the goal.
   Confirm whether the user wants explanation, code review, or implementation/refactor. Establish the list of states and transitions up front.
2. Check applicability.
   If there are only two or three states that rarely change, a simple flag or enum + switch may be cleaner. Use State when states are numerous, transitions are complex, or the code is already difficult to maintain.
3. Draw the state machine.
   Map all states as nodes and all transitions as labelled edges. This diagram drives the design.
4. Define the State interface.
   Declare one method per context action/event that can produce different behavior per state. Avoid methods that make no sense in any state.
5. Implement Concrete State classes.
   One class per state. Each implements the State interface. State-specific logic lives here. Common behavior across states can go in an abstract base state class.
6. Add the context reference to states (if needed).
   States that need to trigger transitions or call context services hold a backreference to the context object. Pass context via the constructor or method parameter.
7. Refactor the Context.
   Replace all state conditionals with delegation: `state.handleAction()`. Add a `setState(state)` method. The context no longer contains any state-branching logic.
8. Assign transition responsibility.
   Decide per transition whether the context, the current state, or the client initiates it. The most common choice: states initiate transitions (`context.setState(new NextState(context))`).
9. Validate behavior.
   Confirm each state behaves correctly in isolation. Confirm adding a new state requires only a new state class and wiring of transitions, zero changes to existing states or context methods.
10. Explain tradeoffs.
    Call out added class count, the risk of tight coupling between states, and the difference from Strategy.

## Decision Branches
- If behavior varies per mode but states are completely independent and know nothing of each other:
  Consider Strategy — strategies are interchangeable algorithms, not a state machine. State objects may know each other and trigger transitions; strategies do not.
- If there are only 2–3 stable states with minimal branching:
  A simple `enum` + `switch` is clearer and sufficient; avoid pattern overhead.
- If transitions must be persisted or serialized:
  Store the state name/enum value; reconstruct the concrete state object on load.
- If states share significant behavior:
  Extract an abstract base state class or use mixins/traits to avoid duplication.
- If multiple objects share the same stateless behavior for a given state:
  Flyweight the state objects — share one instance of each state class across all contexts.

## Output Contract
When responding, provide:
1. A suitability verdict (State or a simpler conditional alternative).
2. A state machine diagram (text or ASCII) showing states and transitions.
3. Named roles mapped to the user's domain (Context, State interface, Concrete States).
4. Transition ownership decision (who calls `setState`).
5. A minimal code sketch in the user's language showing context delegation and one state transition.
6. Validation checklist.

## Quality Checks
- Context contains zero state-branching conditionals — all branching is in state classes.
- Each Concrete State class handles one state only.
- Adding a new state requires zero changes to existing state classes or context methods.
- State interface methods are meaningful for every concrete state (no empty/no-op implementations unless intentional).
- Transitions are traceable: every `setState` call has a clear owner and trigger.
- Context's public API is unchanged after the refactor.

## Common Mistakes To Prevent
- Leaving residual `if (state == X)` checks in the context after extracting state classes.
- States calling each other directly (bypassing the context) — breaks the state machine contract.
- Defining state interface methods that only make sense in one state (violates interface segregation).
- Confusing State with Strategy: in State, concrete states know about transitions and each other; in Strategy, concrete strategies are completely independent.
- Forgetting to initialize the context with a valid starting state object.
- Circular state transitions with no exit condition (infinite loop).

## References
- Detailed guide: [REFERENCE.md](./REFERENCE.md)
- Worked examples: [EXAMPLES.md](./EXAMPLES.md)
