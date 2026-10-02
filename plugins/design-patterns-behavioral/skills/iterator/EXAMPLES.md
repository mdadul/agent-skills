# Iterator Examples

## 1) Social Graph Iterator (Canonical — Lazy Remote Fetch)

### Context
A social network exposes friends and coworkers of a profile. The underlying data is fetched via API. Clients must not know whether data comes from a cache, REST call, or graph DB.

### Design
- **Iterator interface**: `ProfileIterator` — `hasNext(): boolean`, `getNext(): Profile`.
- **Concrete Iterator**: `FacebookIterator` — stores `profileId`, `type`, `currentPosition`, lazy-loaded `cache`.
- **Collection interface**: `SocialNetwork` — `createFriendsIterator(id)`, `createCoworkersIterator(id)`.
- **Concrete Collection**: `Facebook`, `LinkedIn` — each implements both factory methods.
- **Client**: `SocialSpammer.send(iterator, message)` — calls `hasNext()` / `getNext()` in a loop; works identically regardless of which network or traversal type is passed.

```typescript
interface ProfileIterator {
    hasNext(): boolean;
    getNext(): Profile;
}

class FacebookIterator implements ProfileIterator {
    private cache: Profile[] | null = null;
    private position = 0;

    constructor(
        private facebook: Facebook,
        private profileId: string,
        private type: "friends" | "coworkers"
    ) {}

    private lazyLoad() {
        if (!this.cache)
            this.cache = this.facebook.fetchProfiles(this.profileId, this.type);
    }

    hasNext(): boolean {
        this.lazyLoad();
        return this.position < this.cache!.length;
    }

    getNext(): Profile {
        return this.cache![this.position++];
    }
}
```

Two callers can hold separate `FacebookIterator` instances on the same profile — independent `position` fields, no interference.

---

## 2) Binary Tree — Depth-First and Breadth-First Iterators

### Context
A `BinaryTree<T>` should support both depth-first (in-order) and breadth-first traversal without the client managing a stack or queue.

### Design
- **Collection interface**: `createDepthFirstIterator(): Iterator<T>`, `createBreadthFirstIterator(): Iterator<T>`.
- **Concrete Iterators**: `DepthFirstIterator<T>` (stack-based), `BreadthFirstIterator<T>` (queue-based).

```typescript
class DepthFirstIterator<T> implements Iterator<T> {
    private stack: TreeNode<T>[];

    constructor(root: TreeNode<T> | null) {
        this.stack = root ? [root] : [];
    }

    hasNext(): boolean { return this.stack.length > 0; }

    next(): T {
        const node = this.stack.pop()!;
        if (node.right) this.stack.push(node.right);
        if (node.left)  this.stack.push(node.left);
        return node.value;
    }
}
```

Switching traversal algorithm → swap iterator at the `createIterator()` call site; client loop unchanged.

---

## 3) TypeScript — Native `Symbol.iterator` Protocol

### Context
A custom `Range` collection should work with `for...of`, spread, and destructuring.

```typescript
class Range implements Iterable<number> {
    constructor(private from: number, private to: number) {}

    [Symbol.iterator](): Iterator<number> {
        let current = this.from;
        const last = this.to;
        return {
            next(): IteratorResult<number> {
                return current <= last
                    ? { value: current++, done: false }
                    : { value: undefined as any, done: true };
            }
        };
    }
}

// Client — no knowledge of internal state
for (const n of new Range(1, 5)) console.log(n);  // 1 2 3 4 5
const nums = [...new Range(1, 3)];                 // [1, 2, 3]
```

Implementing `Symbol.iterator` gives access to the full JavaScript iteration ecosystem for free.

---

## 4) Python Generator as Iterator

### Context
A `FileLineCollection` reads a large log file line by line. Loading all lines into memory is wasteful.

```python
class FileLineCollection:
    def __init__(self, path: str):
        self._path = path

    def __iter__(self):
        with open(self._path) as f:
            for line in f:
                yield line.rstrip('\n')  # lazy — one line at a time

# Client
for line in FileLineCollection("/var/log/app.log"):
    process(line)  # never loads the full file into memory
```

Python generators implement `__iter__` and `__next__` automatically — idiomatic lazy Iterator.

---

## 5) Parallel Iteration Proof

```typescript
const tree = new BinaryTree([5, 3, 7, 1, 4]);

const iter1 = tree.createDepthFirstIterator();
const iter2 = tree.createDepthFirstIterator();

iter1.next();  // advances iter1 to position 1
iter1.next();  // advances iter1 to position 2

// iter2 is still at position 0 — completely independent
console.log(iter2.next().value);  // always first element regardless of iter1
```

Each iterator owns its traversal state. The collection is not modified.

---

## 6) When NOT to Use Iterator

| Scenario | Better approach |
|---|---|
| Simple array, single traversal | `for` loop or `.forEach()` |
| Built-in `.map()` / `.filter()` covers the need | Use standard library |
| One-off traversal inside a single function | Inline loop — no need for a class |
| Async pagination with `await` per page | Async generator (`async function*`) |

---

## 7) Quick Evaluation Prompts
- "How do I implement a custom iterator for my binary tree in TypeScript?"
- "Should I implement `Symbol.iterator` or create a separate iterator class?"
- "How can two callers iterate the same collection at the same time?"
- "Show me a lazy iterator for a large file in Python."
- "When is Iterator overkill and a plain loop is better?"
