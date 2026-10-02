---
name: memento
description: Teach and apply the Memento (Snapshot) pattern to save and restore object state without breaking encapsulation. Use when users ask about Memento, Snapshot, undo/redo, state history, transaction rollback, or comparing Memento to Prototype or Command.
---

# Memento

Also known as: Snapshot

## Purpose
Help the user decide whether Memento is appropriate, then design and apply it so the originator object owns snapshot creation and restoration — without leaking private state to any other class.

## When To Use
- User asks to explain the Memento or Snapshot pattern.
- An undo/redo mechanism is needed and the object's state must be captured without exposing private fields.
- Transactional rollback is required: save state before a risky operation, restore on failure.
- An object's state history must be tracked externally (caretaker/history) without coupling that history to the originator's internals.
- Iteration position needs to be saved and restored (combined with Iterator).

## Inputs
- The originator class and which fields constitute its restorable state.
- The granularity of snapshots: full state each time, or incremental/diff-based.
- The caretaker type: history stack (undo/redo), command objects, transaction manager, etc.
- Memory constraints (drives snapshot size strategy and history limits).
- Language support for nested classes, access modifiers, or serialization.

## Workflow
1. Clarify the goal.
   Confirm whether the user wants explanation, code review, or implementation/refactor. Confirm undo/redo, rollback, or point-in-time restore is the actual requirement.
2. Check applicability.
   If the object is simple with only public state, Prototype (clone) may be a lighter alternative. Use Memento when encapsulation must be preserved and state includes private fields.
3. Identify the Originator.
   Confirm which class owns the state to be snapshotted. It must be the one producing and consuming mementos — no outsider should read raw state from the memento.
4. Design the Memento.
   Mirror the originator's restorable fields as immutable fields set only in the constructor. No setters. Expose only metadata (timestamp, label) to outsiders — never the raw state.
5. Choose the encapsulation strategy.
   - Nested class (Java/C#/C++): memento is private to originator, caretaker holds an opaque reference.
   - Intermediate interface: caretaker holds `IMemento` (metadata only); originator casts to concrete class.
   - Memento-holds-originator reference: `restore()` lives on the memento; originator provides setters.
6. Add `save()` / `createSnapshot()` to the Originator.
   Returns a new Memento capturing current state. The originator passes its own fields directly.
7. Add `restore(memento)` to the Originator (or on the Memento itself).
   Applies the captured state back to the originator's fields.
8. Implement the Caretaker.
   Manages when to save and when to restore. Typically a stack for undo/redo or a list for full history. The caretaker never reads memento contents — only stores and passes them back.
9. Validate behavior.
   Confirm private fields are inaccessible to the caretaker. Confirm save → mutate → restore returns the originator to the exact prior state. Confirm multiple independent mementos coexist.
10. Explain tradeoffs.
    Call out RAM usage for frequent snapshots, lifecycle management of old mementos, and the language-specific encapsulation caveats.

## Decision Branches
- If the object is simple and has no complex private state or external references:
  Consider Prototype (clone) instead — lighter and sufficient for value-like objects.
- If undo requires both state restore AND operation reversal:
  Combine Memento with Command — Command performs/undoes the operation; Memento carries the state snapshot.
- If the snapshot is taken per-command (one memento per command object):
  Store the memento on the command object itself (command acts as caretaker).
- If only iteration position needs saving:
  Combine with Iterator — Memento snapshots the iterator's `currentPosition`.
- If memory is tight:
  Consider diff-based snapshots (store only changes), capped history, or weak references for old mementos.
- If the language doesn't enforce access restrictions at runtime (PHP, Python, JS):
  Document the convention clearly; rely on naming (`_private`) or serialization to opaque types.

## Output Contract
When responding, provide:
1. A suitability verdict (Memento or a simpler alternative like Prototype).
2. Named roles mapped to the user's domain (Originator, Memento, Caretaker).
3. The chosen encapsulation strategy and why it fits the language.
4. Originator `save()` and `restore()` method sketches.
5. Caretaker structure (stack, history list, or per-command storage).
6. Memory management guidance if frequent snapshots are expected.
7. Validation checklist.

## Quality Checks
- Originator produces and consumes mementos — no other class reads raw state from memento.
- Memento is immutable: constructor-only initialization, no setters, no public state fields.
- Caretaker holds mementos but cannot read their internal state.
- `restore(memento)` returns the originator to the exact state captured at `save()` time.
- Multiple mementos can coexist without interfering with each other.
- Old mementos are cleaned up (bounded history or explicit caretaker lifecycle management).

## Common Mistakes To Prevent
- Exposing originator state via public memento fields (breaks encapsulation — defeats the pattern).
- Storing mementos with references to mutable objects instead of deep copies (snapshot becomes stale).
- Caretaker logic that reads or interprets memento contents (should treat mementos as opaque).
- Unbounded history growth consuming all available RAM.
- Assuming shallow copy is sufficient when originator state contains nested mutable objects.
- Using Memento when the state is trivially simple — Prototype/clone is cleaner there.

## References
- Detailed guide: [REFERENCE.md](./REFERENCE.md)
- Worked examples: [EXAMPLES.md](./EXAMPLES.md)
