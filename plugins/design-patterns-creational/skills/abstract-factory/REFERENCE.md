# Abstract Factory Reference

## Intent
Abstract Factory provides an interface for creating families of related objects without specifying concrete classes.

## Problem Signal
- Client code creates several related product types directly (`new` calls spread across code).
- Product variants must stay consistent (theme/platform/vendor), but mismatches occur.
- Adding a new variant requires edits in many construction sites.

## Solution Shape
1. Define abstract product interfaces for each product type.
2. Define an abstract factory with one creation method per product type.
3. Implement concrete factories, one per variant.
4. Keep client code dependent on abstract interfaces only.
5. Select concrete factory once at initialization, then inject/use it.

## Roles
- Abstract Product: Interface for each product type.
- Concrete Product: Variant-specific implementation of an abstract product.
- Abstract Factory: Interface exposing creation methods for all product types.
- Concrete Factory: Creates a coherent variant family.
- Client: Consumes factory and products through abstractions only.

## Applicability
Use when:
- You need families of related objects that must remain compatible.
- You want to support unknown/future variants without changing client logic.
- You need to separate creation from usage across multiple product types.

Avoid when:
- There is only one product type with limited variability.
- The product matrix changes heavily by product type (can cause class explosion).

## Refactor Recipe
1. Build matrix: product types x variants.
2. Create abstract products for each product type.
3. Create abstract factory API covering all product types.
4. Implement concrete factories per variant.
5. Centralize factory selection in startup/composition root.
6. Replace direct constructors with factory calls.

## Validation Checklist
- Products from one factory are variant-consistent and collaborate correctly.
- Client code does not reference concrete products or factories.
- New variant can be introduced by adding concrete products + factory only.
- Existing behavior remains unchanged after migration.

## Pros
- Enforces compatibility among related products.
- Reduces coupling to concrete classes.
- Supports Open/Closed extension for new variants.

## Cons
- Adds interfaces/classes and upfront structure.
- Adding new product types requires updates across all factories.

## Relationship Notes
- Often evolves from multiple Factory Methods.
- Differs from Builder: Abstract Factory returns products immediately.
- Can coordinate with Bridge when abstraction-implementation compatibility is constrained.
