# Command Reference

## Intent
Command encapsulates a request as an object, separating invocation from execution and enabling queueing, logging, scheduling, and undoable operations.

## Problem Signal
- UI/event layers are tightly coupled to business operation methods.
- Same action logic is duplicated across multiple invokers.
- Operations need delayed execution, retries, or remote dispatch.
- Undo/redo requirements are hard to retrofit into direct-call code.

## Solution Shape
1. Define command interface (`execute`, optional `undo`).
2. Implement concrete commands with receiver + arguments captured as fields.
3. Have invokers trigger commands through interface only.
4. Add optional infrastructure (history stack, queue, scheduler, bus).
5. Keep receivers focused on domain logic while commands orchestrate invocation details.

## Roles
- Invoker/Sender: Initiates command execution.
- Command Interface: Common execution contract.
- Concrete Command: Encapsulates receiver and operation payload.
- Receiver: Executes actual business behavior.
- Client: Wires invokers, commands, and receivers.
- Command History (optional): Stores executed commands for undo/redo.

## Applicability
Use when:
- Operations should be first-class values passed/stored/executed later.
- Invokers should be independent from receiver implementations.
- You need operational history, replay, queueing, or remote execution.

Avoid when:
- Simple direct calls are sufficient and no extensibility/execution control is needed.
- Extra object layer adds overhead with no tangible benefit.

## Undo Models
- Snapshot-based: command stores prior state (or memento) then restores on undo.
- Inverse-operation: command executes mathematically/logically opposite action.
- Hybrid: snapshot for complex mutable aggregates, inverse for lightweight operations.

## Refactor Recipe
1. Locate direct invoker-to-receiver calls.
2. Introduce command interface and one concrete command for one operation.
3. Move operation payload and receiver into command constructor.
4. Update invoker to execute command object.
5. Expand to remaining operations.
6. Add history/queue/scheduler only where needed.
7. Add undo/redo behavior for mutating commands.

## Validation Checklist
- Invoker can switch command implementations without code changes.
- Commands execute correctly with serialized/deserialized payloads if required.
- Undo restores expected state and redo reapplies correctly.
- Non-mutating commands are not polluting mutation history.
- Command failures surface through a stable error model.

## Pros
- Decouples invocation from execution.
- Supports Open/Closed extension with new command classes.
- Enables undo/redo, history, queueing, and remote dispatch patterns.

## Cons
- Adds indirection and class count.
- Undo semantics can be expensive for large state snapshots.

## Relationship Notes
- Strategy swaps algorithms; Command encapsulates executable requests with payload/history use cases.
- CoR routes requests through handlers; Command packages the request itself.
- Memento often complements Command for snapshot-based undo.
- Composite can represent macro commands composed of child commands.
