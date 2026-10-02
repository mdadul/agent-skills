---
name: visitor
description: Teach and apply the Visitor pattern to add new operations to a stable element hierarchy without modifying element classes, using double dispatch. Use when users ask about Visitor, double dispatch, adding behavior to sealed/frozen class hierarchies, separating algorithms from object structures, or combining Visitor with Composite or Iterator.
---

# Visitor

## Purpose
Help the user decide whether Visitor is appropriate, then design and apply it to extract new operations into separate visitor classes — keeping element classes clean and unchanged — while using the double dispatch technique to route to the correct per-type implementation.

## When To Use
- User asks to explain the Visitor pattern.
- A new operation must be added to a class hierarchy whose classes cannot or should not be modified (production, third-party, frozen API).
- Multiple unrelated operations are accumulating inside element classes, polluting their primary responsibility.
- An operation needs a different implementation for each concrete element type in a heterogeneous collection.
- The operation makes sense only for some element types — visitor methods for irrelevant types can be left as no-ops.
- Traversal over a Composite tree must trigger per-type logic at each node.

## Inputs
- The element class hierarchy (stable, finite set of concrete types).
- The new operation(s) to add (each becomes a Concrete Visitor).
- Whether the hierarchy is open to adding `accept(Visitor)` to each class (the one required change).
- Language support for method overloading or pattern matching (affects dispatch approach).
- Access level of element fields the operation needs.

## Workflow
1. Clarify the goal.
   Confirm whether the user wants explanation, code review, or implementation/refactor. Confirm the element hierarchy is stable — Visitor's main cost is updating all visitors when a new element type is added.
2. Check applicability.
   Visitor is strong when element types are stable and operations change frequently. If element types change often, the cost of updating every visitor per new element may outweigh the benefit.
3. Declare the Visitor interface.
   One `visit(ConcreteElement)` method per concrete element class. Use overloading if the language supports it; use distinct names (`visitDot`, `visitCircle`) otherwise.
4. Add `accept(Visitor)` to the Element interface.
   One method: `accept(v: Visitor)`. This is the only required change to existing element classes.
5. Implement `accept` in each Concrete Element.
   Each calls the specific visitor method that matches its own type: `v.visitDot(this)` in `Dot`, `v.visitCircle(this)` in `Circle`. This is double dispatch — the element's type routes to the correct visitor method.
6. Implement Concrete Visitors.
   One class per operation. Each implements all visitor methods. Intermediate state needed during traversal (e.g., accumulating XML string) lives in the visitor instance.
7. Traverse and dispatch.
   Client iterates the element collection, calls `element.accept(visitor)` on each. No `instanceof` checks or casts needed in client code.
8. Validate behavior.
   Confirm adding a new operation requires only a new Concrete Visitor class — zero element class changes. Confirm the client loop is free of type-conditional logic.
9. Explain tradeoffs.
    Call out the tradeoff: new operations are cheap; new element types are expensive (all visitors must be updated).

## Decision Branches
- If the element hierarchy grows frequently (new types added often):
  Visitor is a poor fit — every new element type forces updates to all existing visitors. Consider putting behavior in the elements via virtual dispatch instead.
- If only one operation is needed and it's closely related to the element's domain:
  Put it directly in the element class; Visitor overhead is unjustified.
- If the language has pattern matching / discriminated unions (Kotlin `when`, TypeScript discriminated union, Haskell):
  Pattern matching can replace Visitor cleanly. Evaluate before introducing the full pattern.
- If traversal over a Composite tree is involved:
  Combine with Iterator (Iterator traverses, Visitor operates per node) or let the Composite's `accept` method delegate to children recursively.
- If the operation is a command with undo/redo:
  Combine with Command — Visitor is a generalization of Command that works across multiple element types.

## Output Contract
When responding, provide:
1. A suitability verdict — specifically whether the element hierarchy is stable enough.
2. Named roles mapped to the user's domain (Visitor interface methods, Element interface, Concrete Elements, Concrete Visitors).
3. The double dispatch explanation: why `accept` + `visit` is needed (vs. instanceof).
4. A minimal code sketch in the user's language showing at least two element types and one visitor.
5. Validation checklist.

## Quality Checks
- Element classes contain only `accept(v: Visitor)` as a visitor-related change — no operation logic.
- Each Concrete Element's `accept` calls the visitor method specific to its own type.
- Client loop uses `element.accept(visitor)` with no `instanceof` or type casts.
- Adding a new operation requires only a new Concrete Visitor class.
- Visitor interface has exactly one method per concrete element type.
- Visitor accumulates intermediate state in instance fields, not in elements.

## Common Mistakes To Prevent
- Having `accept` in the base element call a generic `visitor.visit(this)` — loses type information and breaks double dispatch.
- Calling `v.visitCircle(this)` from `Dot.accept` — dispatches to the wrong visitor method.
- Adding operation logic directly to element `accept` methods (defeats the purpose).
- Applying Visitor to a hierarchy that gains new element types frequently — updating all visitors becomes a maintenance burden.
- Forgetting to implement a visitor method for one element type — compiler may not catch no-ops.
- Confusing Visitor with Strategy: Visitor operates across a heterogeneous type hierarchy using double dispatch; Strategy swaps one algorithm on a single context type.

## References
- Detailed guide: [REFERENCE.md](./REFERENCE.md)
- Worked examples: [EXAMPLES.md](./EXAMPLES.md)
