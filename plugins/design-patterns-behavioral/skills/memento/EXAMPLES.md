# Memento Examples

## 1) Text Editor Undo (Canonical — Command + Memento)

### Design
- **Originator**: `Editor` — owns `text`, `curX`, `curY`, `selectionWidth`.
- **Memento**: `EditorSnapshot` — immutable, captures all four fields plus a timestamp label.
- **Caretaker**: `CommandHistory` — stack of `Command` objects; each command stores a snapshot taken before it executed.

```typescript
class Editor {
    private text = "";
    private curX = 0;
    private curY = 0;

    setText(t: string)          { this.text = t; }
    setCursor(x: number, y: number) { this.curX = x; this.curY = y; }

    save(): EditorSnapshot {
        return new EditorSnapshot(this.text, this.curX, this.curY);
    }

    restore(s: EditorSnapshot) {
        this.text = s.getText();
        this.curX = s.getCurX();
        this.curY = s.getCurY();
    }
}

class EditorSnapshot {
    private readonly text: string;
    private readonly curX: number;
    private readonly curY: number;
    private readonly date = new Date();

    constructor(text: string, curX: number, curY: number) {
        this.text = text; this.curX = curX; this.curY = curY;
    }

    // Only metadata is public — raw state only accessible to Editor
    getLabel() { return `Snapshot [${this.date.toISOString()}]`; }

    // Package-private getters — in TypeScript, use naming convention or module boundary
    getText()  { return this.text; }
    getCurX()  { return this.curX; }
    getCurY()  { return this.curY; }
}

class Command {
    private snapshot: EditorSnapshot | null = null;

    constructor(private editor: Editor) {}

    execute(action: () => void) {
        this.snapshot = this.editor.save();  // capture before
        action();
    }

    undo() {
        if (this.snapshot) this.editor.restore(this.snapshot);
    }
}
```

Each `Command` is its own caretaker. `CommandHistory` holds a stack of `Command` objects; undo pops and calls `cmd.undo()`.

---

## 2) Database Transaction Rollback

### Context
A service executes a multi-step operation on an in-memory aggregate. If any step fails, the aggregate must be rolled back to its pre-operation state.

```typescript
class OrderAggregate {
    private items: OrderItem[] = [];
    private status: OrderStatus = "draft";
    private total = 0;

    save(): OrderSnapshot {
        // Deep copy items array — mutable nested objects must be cloned
        return new OrderSnapshot(
            [...this.items.map(i => ({ ...i }))],
            this.status,
            this.total
        );
    }

    restore(snapshot: OrderSnapshot) {
        this.items  = snapshot.getItems();
        this.status = snapshot.getStatus();
        this.total  = snapshot.getTotal();
    }
}

async function processOrder(order: OrderAggregate, payment: Payment) {
    const checkpoint = order.save();
    try {
        order.addItem(payment.item);
        order.applyDiscount(payment.coupon);
        order.confirm();
        await paymentGateway.charge(payment);
    } catch (e) {
        order.restore(checkpoint);  // rollback to pre-operation state
        throw e;
    }
}
```

The caretaker here is the `processOrder` function itself — it saves before and restores on error.

---

## 3) Game Save Point

### Context
A game saves the player's state at a checkpoint. The player can die and respawn at the last checkpoint.

```typescript
class PlayerState {
    constructor(
        private hp: number,
        private x: number,
        private y: number,
        private inventory: Item[]
    ) {}

    save(): PlayerMemento {
        return new PlayerMemento(this.hp, this.x, this.y, [...this.inventory]);
    }

    restore(m: PlayerMemento) {
        this.hp        = m.hp;
        this.x         = m.x;
        this.y         = m.y;
        this.inventory = [...m.inventory];
    }
}

class CheckpointManager {
    private saves: PlayerMemento[] = [];

    checkpoint(player: PlayerState) {
        this.saves.push(player.save());
    }

    respawn(player: PlayerState) {
        const last = this.saves.at(-1);
        if (last) player.restore(last);
    }
}
```

---

## 4) Iterator Position Snapshot (Memento + Iterator)

### Context
A complex tree traversal needs to be paused, saved, and resumed later.

```typescript
class TreeIterator {
    private stack: TreeNode[];

    constructor(root: TreeNode) { this.stack = [root]; }

    // Save current traversal position
    save(): IteratorMemento {
        return new IteratorMemento([...this.stack]);  // snapshot the stack
    }

    restore(m: IteratorMemento) {
        this.stack = [...m.getStack()];
    }

    next(): TreeNode { /* pop stack, push children */ }
    hasNext(): boolean { return this.stack.length > 0; }
}

class IteratorMemento {
    constructor(private readonly stack: TreeNode[]) {}
    getStack() { return this.stack; }
}
```

---

## 5) Encapsulation Strategy Comparison

| Strategy | Languages | Caretaker sees | Enforcement |
|---|---|---|---|
| Nested private class | Java, C#, C++ | Opaque reference only | Compiler-enforced |
| Intermediate interface | All | Metadata interface only | Convention + compile-time type |
| Memento holds originator ref | All | Calls `restore()` only | Convention |
| Serialization to opaque bytes | All | `byte[]` / `string` blob | Runtime — unreadable but not type-safe |

---

## 6) Memory Cost Estimate

```
Full snapshot cost = fields_per_object × bytes_per_field × snapshot_count

Example: text editor
  - text (avg 5KB) + cursor (8 bytes) + selection (4 bytes) ≈ 5KB per snapshot
  - 100 undo levels × 5KB = 500KB  ← acceptable
  - 1000 undo levels × 500KB (large doc) = 500MB ← needs bounded history or diffs
```

**Mitigations**: cap history at N, use diff/delta snapshots, serialize old mementos to disk.

---

## 7) Quick Evaluation Prompts
- "How do I implement undo/redo without exposing private fields?"
- "Should I use Memento or just clone the object for rollback?"
- "Show me Memento + Command for a text editor in TypeScript."
- "How do I prevent the caretaker from reading the snapshot's internal state?"
- "My snapshots are consuming too much RAM — what are my options?"
