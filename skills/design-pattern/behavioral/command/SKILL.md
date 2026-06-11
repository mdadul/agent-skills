---
name: command
description: Teach and apply the Command pattern to encapsulate requests as objects for decoupled invocation, queuing, scheduling, logging, and undo/redo workflows. Use when users ask about Command, Action, Transaction, invoker-receiver decoupling, reversible operations, or comparing Command with Strategy, Chain of Responsibility, or Memento.
---

# Command

Also known as: Action, Transaction

## Purpose
Help the user decide whether Command is appropriate, then design and implement request objects that decouple senders from receivers while enabling history, deferred execution, and reversibility.

## When To Use
- User asks to explain Command pattern.
- UI/event senders trigger operations but should not depend on business receiver details.
- Same operation is triggered by multiple invokers (buttons, menus, shortcuts, jobs).
- Operations must be queued, scheduled, logged, serialized, or dispatched remotely.
- Undo/redo or operation history is required.
- System needs composable macro commands built from smaller commands.

## Inputs
- Sender/invoker types and operation triggers.
- Receiver classes and domain operations to execute.
- Command payload requirements (arguments, context, metadata).
- Execution model (sync, async, queued, scheduled, distributed).
- Reversibility requirements (snapshot-based undo vs inverse operation).

## Workflow
1. Clarify the goal.
   Confirm whether the user needs conceptual guidance, architecture review, or implementation/refactor.
2. Check applicability.
   Use Command when operations must be first-class objects and invoker-receiver coupling should be minimized.
3. Define command contract.
   Create a stable command interface (typically `execute()`, optionally `undo()`).
4. Model concrete commands.
   Encapsulate receiver reference plus all request parameters in command objects.
5. Refactor invokers.
   Replace direct receiver calls with command execution via interface.
6. Add execution infrastructure.
   Introduce dispatcher/history/queue/scheduler depending on requirements.
7. Implement reversibility.
   Use backups/mementos or inverse operations for undoable commands.
8. Support composition.
   Create macro/composite commands where workflows need grouped atomic actions.
9. Validate behavior.
   Verify parity with previous behavior, correct history semantics, and robust replay/undo.

## Decision Branches
- If only algorithm swapping is needed within one context:
  Prefer Strategy.
- If request should be handled by one of many handlers in sequence:
  Prefer Chain of Responsibility.
- If the main requirement is state snapshots without operation objects:
  Prefer Memento.
- If invoker can directly call one stable receiver method and no history/queue is needed:
  Prefer direct call for simplicity.
- If operations must be batched as one user action:
  Use macro command composed of child commands.

## Output Contract
When responding, provide:
1. A suitability verdict (Command or simpler alternative).
2. Role mapping (Invoker/Sender, Command Interface, Concrete Command(s), Receiver, Client, optional History/Dispatcher).
3. Incremental migration plan from direct calls to command objects.
4. Minimal code sketch in the user's language showing one invoker and one receiver decoupled by command.
5. Undo/redo strategy recommendation when relevant.
6. Validation checklist proving execution correctness and decoupling.

## Quality Checks
- Invokers depend only on command interface, not concrete receiver APIs.
- Concrete commands contain complete execution data and receiver references.
- Command execution is deterministic and side effects are explicit.
- History tracks only state-changing commands where required.
- Undo/redo semantics are well-defined and tested.
- Adding new command types does not require invoker changes.

## Common Mistakes To Prevent
- Letting invokers create receiver-specific logic directly.
- Commands with hidden ambient dependencies instead of explicit payload fields.
- Recording non-mutating commands in undo history without purpose.
- Inverse operations that are not true reversals.
- Conflating Command with Strategy and losing queue/history benefits.

## References
- Detailed guide: [REFERENCE.md](./REFERENCE.md)
- Worked examples: [EXAMPLES.md](./EXAMPLES.md)
