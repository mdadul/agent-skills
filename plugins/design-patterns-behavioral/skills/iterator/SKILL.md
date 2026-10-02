---
name: iterator
description: Teach and apply the Iterator pattern to traverse collections without exposing their internal structure. Use when users ask about Iterator, custom traversal algorithms, decoupling traversal from collections, parallel iteration, lazy iteration, or comparing Iterator to for-each loops and language-native iterables.
---

# Iterator

## Purpose
Help the user decide whether Iterator is appropriate, then design and apply it to extract traversal logic out of collections or client code and into a reusable, independently stateful iterator object.

## When To Use
- User asks to explain the Iterator pattern.
- A collection has a complex internal structure (tree, graph, custom linked structure) and clients shouldn't need to understand it to traverse it.
- Multiple traversal algorithms are needed for the same collection (depth-first, breadth-first, filtered, reversed).
- The same collection must be traversed by multiple callers simultaneously, each with independent state.
- Traversal code is duplicated across client code and should be centralized.
- Iteration must be lazy (elements fetched on demand rather than all at once).
- The collection type is unknown at design time and clients must work with any conforming collection.

## Inputs
- The collection type and its underlying data structure.
- The traversal algorithm(s) needed (linear, tree walk, filtered, lazy-loaded, etc.).
- Whether multiple simultaneous iterations over the same collection are required.
- Whether the language already provides an iterator protocol (`Iterable`/`Iterator` in Java, `Symbol.iterator` in JS, `__iter__`/`__next__` in Python, `IEnumerable` in C#).
- Performance constraints (lazy vs. eager, large datasets, remote data).

## Workflow
1. Clarify the goal.
   Confirm whether the user wants explanation, code review, or implementation/refactor. Check if the language has a built-in iterator protocol that should be implemented rather than invented.
2. Check applicability.
   If the collection is a simple flat list and only one traversal is needed, a `for` loop is fine. Apply Iterator when traversal complexity, multiple algorithms, or parallel iteration justifies the indirection.
3. Define the Iterator interface.
   Declare at minimum: `hasNext(): boolean`, `next(): T`. Add `reset()`, `peek()`, or `current()` if the use case requires them.
4. Define the Collection (Iterable) interface.
   Declare a factory method: `createIterator(): Iterator<T>`. Add additional methods for alternate traversal types if needed.
5. Implement Concrete Iterator(s).
   Each holds a reference to the collection and all traversal state (current position, visited set, etc.). Traversal state is never stored in the collection.
6. Implement `createIterator()` in Concrete Collections.
   Returns a new iterator instance linked to the collection. Each call produces an independent iterator with fresh state.
7. Update client code.
   Replace direct collection access with iterator usage. Clients obtain iterators from collections and call `hasNext()` / `next()` (or use the language's `for-of` / `foreach` if the iterator protocol is implemented).
8. Validate behavior.
   Confirm two iterators on the same collection are independent. Confirm adding a new traversal algorithm requires only a new iterator class.
9. Explain tradeoffs.
   Call out added class count for simple cases, and the fit with language-native iteration protocols.

## Decision Branches
- If the language has a built-in iterator protocol:
  Implement that protocol (e.g., `Symbol.iterator` in JS/TS, `__iter__` in Python, `IEnumerable` in C#) so the collection works with native `for-of` / `foreach` loops.
- If only one simple traversal is needed for a flat list:
  Skip Iterator — a plain `for` loop or built-in `.forEach()` is clearer and sufficient.
- If traversal must be pausable or resumable across asynchronous operations:
  Consider a generator function / async iterator instead of a hand-rolled class.
- If iteration state needs to be saved and restored:
  Combine with Memento to snapshot the iterator's state.
- If traversal is over a heterogeneous Composite tree with type-specific operations:
  Combine with Visitor — Iterator handles traversal, Visitor handles the per-element operation.
- If the collection creates iterators of different types per subclass:
  Combine with Factory Method — collection subclasses override the iterator factory method.

## Output Contract
When responding, provide:
1. A suitability verdict (Iterator or a simpler alternative).
2. Named roles mapped to the user's domain (Iterator interface, Concrete Iterator, Collection interface, Concrete Collection).
3. Step-by-step plan showing where traversal state moves and what client code changes.
4. A minimal code sketch in the user's language, ideally implementing the language's native iterator protocol.
5. Parallel iteration proof (two iterator instances on the same collection, independent state).
6. Validation checklist.

## Quality Checks
- Iterator holds all traversal state — the collection holds none.
- Each `createIterator()` call produces a fresh iterator with independent state.
- Client code accesses collection elements only through the iterator interface.
- Adding a new traversal algorithm requires only a new Concrete Iterator, not collection changes.
- If the language has a native iterator protocol, the implementation conforms to it.
- Iterator behavior is defined for edge cases: empty collection, single element, exhausted iterator.

## Common Mistakes To Prevent
- Storing traversal position inside the collection (breaks parallel iteration).
- Mutating the collection during iteration without documenting or handling the concurrent-modification case.
- Reinventing a language-native iteration protocol instead of implementing it.
- Returning the collection's internal array directly (exposes internal structure, breaks encapsulation).
- Creating a stateful iterator that is not reset-safe when `reset()` is expected to be called.
- Applying Iterator to trivially simple collections where a plain loop is the right tool.

## References
- Detailed guide: [REFERENCE.md](./REFERENCE.md)
- Worked examples: [EXAMPLES.md](./EXAMPLES.md)
