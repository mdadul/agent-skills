# Command Examples

## 1) Text Editor Undoable Actions

### Context
Toolbar buttons, menu items, and keyboard shortcuts invoke same editor operations.

### Design
- Command interface with `execute()` and optional `undo()`.
- Concrete commands: `CopyCommand`, `CutCommand`, `PasteCommand`, `UndoCommand`.
- Receiver: `Editor`.
- Invoker triggers commands and pushes mutating ones into history.

### Outcome
Multiple UI triggers reuse the same action objects while enabling undo support.

## 2) Job Queue for Email Delivery

### Context
Email sending must be delayed, retried, and processed asynchronously.

### Pattern Use
- `SendEmailCommand` carries recipient/template/payload.
- Queue stores commands for worker execution.
- Retry policy and dead-letter handling operate at command dispatcher layer.

### Outcome
Delivery logic is decoupled from API request lifecycle and can be replayed safely.

## 3) Macro Command for Batch Operations

### Context
User action should apply multiple edits as one atomic operation.

### Pattern Use
- `MacroCommand` holds ordered child commands.
- `execute()` runs all children; `undo()` rolls back in reverse order.

### Outcome
Complex workflows appear as one user-level action with predictable undo semantics.

## 4) Quick Evaluation Prompts
- "Evaluate whether this UI action layer should use Command or direct callbacks."
- "Refactor these invoker-receiver calls into Command with undo history."
- "Design a queueable command model for this background processing pipeline."
- "Compare Command vs Strategy for this operation dispatch requirement."
