# Visitor Reference

## Intent
Visitor lets you add new operations to an existing object structure without modifying the classes of the elements on which it operates.

## Problem Signal
- New operations must be added to a class hierarchy whose source cannot be changed (frozen, third-party, or production-critical).
- Operations unrelated to the elements' primary purpose are accumulating inside element classes.
- Different element types require different implementations of the same operation and the calling code is full of `instanceof` / type switch logic.
- An operation needs to traverse a Composite tree, invoking type-specific logic at each node.

## Double Dispatch Explained
Normal virtual dispatch: `element.method()` — picks the method based on the element's runtime type.
Single problem: `visitor.visit(element)` — picks overload based on compile-time type of `element` (usually `Node` base class), not the runtime concrete type.

**Double dispatch solves this with two virtual calls:**
1. `element.accept(visitor)` — virtual dispatch on element's type routes to the correct `accept` override.
2. Inside `accept`, `visitor.visitDot(this)` — `this` is statically typed as `Dot`, so the correct overload is selected.

Result: the right visitor method for the right element type is called with zero `instanceof` checks.

## Roles
- **Visitor** (interface): Declares one `visit(ConcreteElement)` method per concrete element class. In languages with overloading, all methods may be named `visit`; otherwise use distinct names.
- **Concrete Visitor**: Implements all visitor methods. Each method contains the operation logic for one element type. May accumulate intermediate state in instance fields across multiple `visit` calls.
- **Element** (interface): Declares `accept(v: Visitor)`. This is the only addition required to the existing hierarchy.
- **Concrete Element**: Implements `accept` by calling the visitor method that matches its own type: `v.visitDot(this)` inside `Dot`. No operation logic here.
- **Client**: Creates a visitor, iterates the element collection, calls `element.accept(visitor)` on each. Contains no type-conditional dispatch logic.

## Applicability

Use when:
- The element class hierarchy is **stable** (element types rarely added or removed).
- New operations are added frequently.
- Elements should not carry unrelated behavioral code.
- An operation requires different implementations per element type across a heterogeneous collection.

Avoid when:
- New element types are added frequently — every new type forces updates to all Concrete Visitors.
- The operation is closely related to the element's own domain — just add a virtual method.
- The language supports pattern matching well enough to avoid the ceremony.

## Stability Trade-Off

| Changes frequently | Changes rarely | Prefer |
|---|---|---|
| Operations | Element types | **Visitor** — new ops = new class |
| Element types | Operations | **Virtual methods** — new type = new class + override |

## Refactor Recipe
1. Identify all operations currently scattered in `instanceof` chains or mixed into element classes.
2. Declare a `Visitor` interface with one method per concrete element type.
3. Add `accept(v: Visitor)` to the element base interface/class.
4. Implement `accept` in each Concrete Element: call the matching `visit` method with `this`.
5. Create one Concrete Visitor per operation, implementing all visitor methods.
6. Move per-type operation logic from `instanceof` chains (or element classes) into visitor methods.
7. Replace client-side type checks with `element.accept(visitor)`.

## Encapsulation Consideration
Visitor methods may need access to element private fields. Options:
- Make required fields/methods package-private or internal (least invasive).
- Provide `friend` / nested class access where the language supports it.
- Add accessor methods to elements specifically for visitor use (documents the contract).
- Accept the trade-off: if deep access is required, the element and visitor are inherently coupled.

## Validation Checklist
- `accept` in every Concrete Element calls the visitor method specific to its own type.
- Client code has zero `instanceof` checks or explicit type casts on elements.
- Adding a new operation requires only a new Concrete Visitor class.
- Adding a new element type requires: `accept` implementation in the new class + one new method in every existing Visitor.
- Visitor interface has exactly one method per concrete element type, no more.

## Pros
- Open/Closed: new operations added without touching element classes.
- Single Responsibility: each visitor concentrates one operation; elements stay clean.
- Visitor can accumulate state across a traversal (e.g., collect all nodes of a type).

## Cons
- Adding a new element type breaks all existing visitors (must add a new method to each).
- Visitors may need access to private element state — can compromise encapsulation.
- Double dispatch is non-obvious; requires understanding of the two-step dispatch.

## Relationship Notes
- **Visitor + Composite**: Natural pairing. `CompoundShape.accept` calls `accept` on each child, then `v.visitCompoundShape(this)`. Visitor traverses the whole tree without the client managing recursion.
- **Visitor + Iterator**: Iterator handles traversal order; Visitor handles the per-element operation. Separates "how to walk" from "what to do at each stop."
- **Visitor vs Command**: Command encapsulates one operation on one receiver. Visitor is a generalization — one visitor class holds one operation implemented across many receiver types.
- **Visitor vs Strategy**: Strategy replaces one algorithm on a single context. Visitor defines one operation across an entire type hierarchy using double dispatch.
