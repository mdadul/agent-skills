# Memento Reference

## Intent
Memento captures and externalizes an object's internal state so that the object can be restored to that state later, without violating encapsulation.

## Problem Signal
- An undo/redo or rollback mechanism requires capturing an object's private state.
- Exposing state via getters/public fields to enable snapshotting breaks encapsulation and couples external classes to internal structure.
- The snapshot storage class (caretaker/history) should not know what the state means — only when to save and restore.

## Roles
- **Originator**: The object whose state is saved. Creates mementos from its own private fields. Restores its state from a memento. Never delegates snapshot logic to an external class.
- **Memento**: An immutable value object holding a copy of the originator's state. Fields are set once in the constructor. Exposes only metadata (timestamp, label) to outsiders — never the raw state.
- **Caretaker**: Manages the lifecycle of mementos (when to save, when to restore, how many to keep). Treats mementos as opaque — never reads their contents. Typically a history stack or a command object.

## Encapsulation Strategies

### 1. Nested Class (Java / C# / C++)
Memento is a private nested class inside Originator. Only Originator can instantiate it or read its fields. Caretaker holds an opaque `Memento` reference (can't access internals).

```java
class Editor {
    private String text;

    Memento save() { return new Memento(text); }
    void restore(Memento m) { this.text = m.text; }

    static class Memento {
        private final String text;           // private — only Editor can read
        private Memento(String t) { this.text = t; }
    }
}
```

### 2. Intermediate Interface
For languages without nested classes. Caretaker holds `IMemento` (metadata interface). Originator casts to the concrete class to access state.

```typescript
interface IMemento {
    getLabel(): string;
    getDate(): Date;
}

class EditorSnapshot implements IMemento {
    constructor(
        private readonly _text: string,    // hidden from caretaker
        private readonly _date = new Date()
    ) {}
    getLabel() { return `Snapshot at ${this._date}`; }
    getDate()  { return this._date; }
    getText()  { return this._text; }      // only Editor calls this
}
```

### 3. Memento Holds Originator Reference
`restore()` lives on the Memento. Originator provides setters. Memento calls them on restore. Caretaker calls `memento.restore()` without ever seeing state.

## Applicability
Use when:
- Undo/redo, transaction rollback, or point-in-time restore is required.
- Producing a snapshot externally would violate the object's encapsulation.
- The caretaker must be decoupled from the originator's internal structure.

Avoid when:
- The object has no private state worth protecting — Prototype (clone) is simpler.
- Snapshot frequency is so high that RAM consumption is prohibitive (consider diffs instead).
- The language cannot enforce access restrictions at runtime — rely on convention and document clearly.

## Refactor Recipe
1. Identify the originator and the fields that constitute restorable state.
2. Create an immutable Memento class mirroring those fields; constructor-only init; no setters.
3. Apply the appropriate encapsulation strategy for the language.
4. Add `save(): Memento` to the originator (passes its own fields to the constructor).
5. Add `restore(m: Memento)` to the originator (reads from memento; updates own fields).
6. Implement or update the caretaker: push on save, pop+restore on undo.
7. Ensure mementos are deep copies when state contains mutable nested objects.
8. Add a history size limit or eviction policy if unbounded growth is a concern.

## Validation Checklist
- Caretaker cannot read originator state through the memento (opaque reference or interface).
- Memento fields are immutable — no post-construction modification possible.
- `save()` followed by mutations followed by `restore()` returns originator to exact prior state.
- Nested mutable objects in state are deep-copied into the memento, not referenced.
- History has a defined upper bound or eviction strategy.
- Originator's public API is unchanged by introducing Memento.

## Memory Management
| Approach | Use case |
|---|---|
| Full snapshot per operation | Small state, infrequent saves |
| Diff / delta snapshot | Large state, frequent saves |
| Bounded stack (max N mementos) | Known memory budget |
| Weak references for old mementos | GC handles eviction when memory is tight |
| Serialize to disk | Very large state; persistence across sessions |

## Pros
- Snapshots preserve private state without breaking encapsulation.
- Originator code stays clean — caretaker handles history management.
- Multiple independent mementos can coexist.
- Enables undo/redo, rollback, and time-travel debugging.

## Cons
- RAM usage grows with snapshot frequency and state size.
- Caretaker must manage memento lifecycle (creation, retention, cleanup).
- Deep copying nested mutable objects adds complexity.
- Languages without access modifiers (PHP, Python, JS) cannot enforce encapsulation mechanically — convention only.

## Relationship Notes
- **Memento + Command**: Natural pair for undo/redo. Command performs/undoes the operation; Memento stores the state captured before the command executes. Command objects act as caretakers.
- **Memento + Iterator**: Memento snapshots the iterator's `currentPosition`, enabling iteration rollback.
- **Memento vs Prototype**: Prototype clones the whole object. Works if state is simple and all fields are public or the object is a value type. Memento is preferred when encapsulation must be preserved.
- **Memento vs Serialization**: Serialization achieves a similar result but usually requires all fields to be accessible and produces a format suitable for persistence, not just in-memory rollback.
