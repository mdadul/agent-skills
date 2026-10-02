# Iterator Reference

## Intent
Iterator provides a way to sequentially access the elements of an aggregate object without exposing its underlying representation.

## Problem Signal
- Client code must understand and navigate a collection's internal structure to traverse it.
- Traversal algorithms are duplicated across multiple clients.
- A collection needs multiple traversal strategies (depth-first, breadth-first, filtered, reversed).
- Two callers must iterate the same collection simultaneously without interfering.
- The collection type is not known at design time; clients must work against an abstraction.

## Roles
- **Iterator** (interface): Declares `hasNext(): boolean`, `next(): T`, and optionally `reset()`, `current()`, `peek()`.
- **Concrete Iterator**: Implements the traversal algorithm. Stores all state: reference to collection, current position, visited tracking, etc. Never modifies the collection.
- **Collection / Iterable** (interface): Declares the factory method `createIterator(): Iterator<T>`. May declare multiple methods for different traversal types.
- **Concrete Collection**: Manages its data structure. Implements `createIterator()` returning a new, independent iterator instance linked to this collection.
- **Client**: Obtains an iterator from the collection and traverses via `hasNext()` / `next()`, or uses the language's native iteration construct if the protocol is implemented.

## Applicability
Use when:
- Collection's internal structure is complex and should be hidden from clients.
- Multiple traversal algorithms are needed for the same collection.
- Multiple simultaneous independent traversals are required.
- Traversal code is duplicated and should live in one place.
- Client must work uniformly with different collection types.

Avoid when:
- The collection is a simple flat array or list with a single, obvious traversal.
- The language provides sufficient built-in iteration (`map`, `filter`, `for-of`) and no custom algorithm is needed.

## Minimum Iterator Interface
```
hasNext(): boolean   — true if more elements remain
next(): T            — return current element and advance position
```

Optional extensions:
```
reset()              — restart iteration from the beginning
current(): T         — peek at current without advancing
peek(): T            — peek at next without advancing
```

## Language-Native Protocols

| Language | Protocol to implement |
|---|---|
| JavaScript / TypeScript | `[Symbol.iterator](): Iterator<T>` on the collection |
| Python | `__iter__(self)` returns self; `__next__(self)` returns next or raises `StopIteration` |
| Java | `Iterable<T>` with `iterator(): Iterator<T>`; `Iterator<T>` with `hasNext()` and `next()` |
| C# | `IEnumerable<T>` with `GetEnumerator(): IEnumerator<T>` |
| Go | No formal protocol; idiomatic: `Next() (T, bool)` or `range`-compatible `func` |
| Kotlin | `Iterable<T>` with `iterator(): Iterator<T>` or `operator fun iterator()` |

Implementing the language protocol allows the collection to work with native `for-of` / `foreach` loops, spread operators, destructuring, and library functions.

## Refactor Recipe
1. Identify all locations where client code navigates the collection's internals directly.
2. Declare an Iterator interface (`hasNext`, `next`, any needed extras).
3. Create a Concrete Iterator that encapsulates all traversal state.
4. Move traversal algorithms from the collection (or clients) into Concrete Iterators.
5. Add `createIterator()` to the collection; return `new ConcreteIterator(this)`.
6. Implement the language's native iterator protocol if applicable.
7. Replace client-side traversal code with iterator usage.
8. Add edge-case handling: empty collection, exhausted iterator, reset behavior.

## Validation Checklist
- Traversal state is entirely in the iterator — zero traversal state in the collection.
- Two `createIterator()` calls produce independent objects with independent positions.
- Client code uses only the Iterator interface; no access to collection internals.
- New traversal algorithm → new Concrete Iterator only; collection unchanged.
- Language-native protocol implemented if applicable.
- Edge cases covered: empty collection returns `hasNext() = false` immediately.

## Pros
- Single Responsibility: traversal logic extracted from collections and clients.
- Open/Closed: new traversal → new iterator, no collection changes.
- Parallel iteration with independent state per iterator.
- Lazy iteration — elements fetched on demand, not all at once.
- Uniform traversal interface across heterogeneous collection types.

## Cons
- Overkill for simple flat collections.
- May be less efficient than direct indexed access for specialized structures.
- Adds classes (Iterator interface, one Concrete Iterator per algorithm).

## Relationship Notes
- **Iterator + Composite**: Natural fit — an iterator traverses the Composite tree (DFS or BFS) without the client knowing the tree structure.
- **Iterator + Factory Method**: Collection subclasses override `createIterator()` to return different concrete iterator types — each subclass tailors iteration to its structure.
- **Iterator + Memento**: Memento snapshots the iterator's position; allows rewinding iteration to a saved point.
- **Iterator + Visitor**: Iterator handles traversal of a complex structure; Visitor defines what to do at each element. Together they cleanly separate "how to walk" from "what to do at each stop."
