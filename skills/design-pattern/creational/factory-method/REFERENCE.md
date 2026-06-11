# Factory Method Reference

## Intent
Factory Method defines an interface for creating objects in a base creator, while allowing subclasses to decide which concrete product to instantiate.

## Problem Signal
- Client/business logic is coupled to concrete classes via direct `new` calls.
- Adding each new product type requires edits across many places.
- Conditional creation logic keeps expanding (`if`/`switch` by type/environment).

## Solution Shape
1. Define a common Product interface.
2. Define Concrete Products implementing that interface.
3. Add factory method to Creator returning Product.
4. Move creation behind factory method.
5. Override factory method in Concrete Creators to vary product type.

## Roles
- Product: Shared contract for all products.
- Concrete Product: Variant implementation of Product.
- Creator: Owns business flow that uses Product abstraction.
- Concrete Creator: Chooses concrete product by overriding factory method.

## Applicability
Use when:
- Exact object types are not known at design time.
- Framework/library users need extension points.
- Expensive resources may be reused from pools/caches.

Avoid when:
- Only one stable product exists and no extension is expected.
- Inheritance is constrained and strategy/composition would be simpler.

## Refactor Recipe
1. Extract shared product interface.
2. Add empty/default factory method on creator.
3. Replace creator-internal constructor calls with factory method calls.
4. Create concrete creators and override factory method.
5. Remove remaining type-conditionals where possible.
6. If base factory has no default behavior, make it abstract.

## Validation Checklist
- Client depends on Product abstraction only.
- Creator business logic unchanged except product construction path.
- Adding new product requires new product + new creator, not client edits.
- Variant behavior verified by tests.

## Pros
- Reduces coupling between creation and usage.
- Improves extensibility (Open/Closed Principle).
- Centralizes creation logic (Single Responsibility Principle alignment).

## Cons
- Adds class hierarchy complexity.
- Can over-engineer simple creation scenarios.

## Relationship Notes
- Can be a step toward Abstract Factory when multiple related product families are needed.
- Can participate inside Template Method workflows.
- Alternative to Prototype when cloning complexity is undesirable.
