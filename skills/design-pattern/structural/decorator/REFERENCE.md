# Decorator Reference

## Intent
Decorator attaches additional responsibilities to an object dynamically. Decorators provide a flexible alternative to subclassing for extending functionality.

## Problem Signal
- Optional behaviors are being combined through inheritance, creating a subclass explosion (M behaviors → 2ᴹ subclasses).
- Behavior needs to be added or removed from specific instances at runtime.
- A class is `final` (closed to inheritance) but behavior extension is needed.
- Cross-cutting concerns (logging, caching, auth, compression) need to wrap arbitrary components.

## Solution Shape
1. Define a Component interface covering all operations clients depend on.
2. Implement a Concrete Component with default behavior.
3. Create a Base Decorator implementing Component, holding a `wrappee: Component` reference, and delegating all calls to it.
4. Create Concrete Decorators extending Base Decorator, overriding methods to add behavior before or after the delegation call.
5. Client composes a stack by wrapping the Concrete Component in decorators; interacts with the outermost object via the Component interface.

## Roles
- **Component** (interface): Declares the interface both the wrapped object and all wrappers must conform to.
- **Concrete Component**: The base object with default behavior. Decorators wrap this.
- **Base Decorator**: Implements Component. Holds a `wrappee` reference of type Component. Delegates every method — adds no logic of its own.
- **Concrete Decorator**: Extends Base Decorator. Overrides one or more methods to inject behavior before or after calling `super` / `wrappee`.
- **Client**: Creates a concrete component, wraps it in one or more decorators in the desired order, then works exclusively via the Component interface.

## Applicability
Use when:
- Objects need optional behaviors that can be combined in arbitrary combinations.
- Runtime addition or removal of responsibilities is required.
- Subclassing is impractical (class explosion, `final` keyword, no multiple inheritance).
- Cross-cutting concerns must wrap reusable components without coupling them.

Avoid when:
- All instances always need all behaviors — subclass or configure in the constructor instead.
- Decorator order must be enforced but the system cannot guarantee it — document or enforce order explicitly.
- Removing a specific decorator from the middle of an assembled stack is a hard requirement (Decorator doesn't support this natively).

## Refactor Recipe
1. Identify all optional behaviors being added through subclass combinations.
2. Extract a Component interface with all methods clients use.
3. Move base behavior into a Concrete Component implementing that interface.
4. Create a Base Decorator: implement Component, add `wrappee` field, delegate every method.
5. Create one Concrete Decorator per optional behavior, adding pre/post logic around delegation.
6. Delete the old subclass combinations.
7. Update client code to assemble decorator stacks explicitly.

## Validation Checklist
- Every decorator and the Concrete Component implement the same Component interface.
- Base Decorator contains zero business logic — only delegation.
- Each Concrete Decorator adds exactly one concern.
- Adding a new decorator requires no changes to existing components or decorators.
- Removing a decorator from the stack produces a valid, functional object.
- Client code does not downcast to a concrete decorator type (breaks transparency).

## Ordering Notes
Decorator order matters when behaviors are not commutative:
- `Encrypt(Compress(file))` → compress first, then encrypt on write; decrypt first, then decompress on read.
- `Compress(Encrypt(file))` → encrypt first, then compress — less effective, possibly incorrect.
Document stack order requirements wherever order affects correctness.

## Pros
- Objects get new behaviors at runtime without subclassing.
- Single Responsibility: each decorator handles one concern.
- Open/Closed: new behaviors added by new decorators, not by modifying existing classes.
- Arbitrary combinations without class explosion.

## Cons
- Hard to remove a specific decorator from the middle of a stack.
- Order-sensitive stacks are a subtle source of bugs.
- Initial wiring code (stacking decorators) can be verbose or ugly.
- Many small objects in the stack may be harder to debug or inspect.

## Relationship Notes
- **Decorator vs Adapter**: Adapter changes the interface; Decorator keeps (or extends) it. Decorator supports recursive composition; Adapter does not.
- **Decorator vs Proxy**: Structurally similar. Proxy manages access and lifecycle of its subject; Decorator enriches behavior. Proxy composition is usually managed by the proxy itself; Decorator stacks are assembled by the client.
- **Decorator vs Composite**: Both use recursive composition. Composite aggregates multiple children (tree); Decorator wraps exactly one child (chain). Decorator adds behavior; Composite combines results. They can cooperate: decorate nodes in a Composite tree.
- **Decorator vs Chain of Responsibility**: Both use recursive composition. CoR handlers can stop the chain; Decorator must always propagate the call. CoR handles requests; Decorator enriches behavior.
- **Decorator vs Strategy**: Strategy swaps the algorithm inside an object (the guts); Decorator adds a layer outside the object (the skin).
- **Decorator + Prototype**: In Composite/Decorator-heavy systems, Prototype can clone assembled stacks instead of rebuilding them.
