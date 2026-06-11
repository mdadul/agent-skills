# Template Method Reference

## Intent
Template Method defines an algorithm skeleton in a base class and lets subclasses customize selected steps without changing algorithm structure.

## Problem Signal
- Multiple classes reimplement the same orchestration with small variations.
- Fixes to algorithm order require repetitive edits across variants.
- Client code branches by concrete type to run similar workflows.
- Common behavior is scattered instead of centralized.

## Solution Shape
1. Move shared orchestration into one template method.
2. Break algorithm into step methods.
3. Make variable steps abstract/overridable.
4. Keep common steps implemented in base class.
5. Add hooks for optional extension points.

## Roles
- Abstract Base Class: Owns template method and shared/default steps.
- Template Method: Ordered algorithm skeleton.
- Concrete Subclass: Implements/overrides selected steps.
- Client: Calls template method polymorphically.

## Applicability
Use when:
- Algorithm sequence is stable but step details vary by subtype.
- You want compile-time enforced extension points via inheritance.
- You need to remove duplication while preserving flow invariants.

Avoid when:
- Behavior must swap dynamically at runtime (Strategy fits better).
- Inheritance is undesirable or disallowed in architecture.

## Step Types
- Abstract step: required implementation in subclass.
- Default step: base behavior reusable by most subclasses.
- Hook: optional no-op extension point before/after key steps.

## Refactor Recipe
1. Identify duplicated algorithm families.
2. Extract canonical step order into base template method.
3. Pull shared step implementations into base class.
4. Mark variable steps abstract/overridable.
5. Introduce hooks only where extension is genuinely needed.
6. Move clients to call template method on base type.

## Validation Checklist
- Template order is fixed and preserved across all subclasses.
- Subclasses implement required abstract steps.
- Default steps reduce duplication without leaking variant assumptions.
- Hooks do not violate base invariants.
- New subclass variants require minimal boilerplate.

## Pros
- Centralizes algorithm flow and reduces duplication.
- Enables controlled extension via subclass overrides.
- Improves consistency and maintainability for related workflows.

## Cons
- Inheritance coupling can limit flexibility.
- Large template hierarchies can become hard to maintain.
- Misused hooks/overrides can erode clarity.

## Relationship Notes
- Strategy: composition-based runtime swapping; Template Method: inheritance-based compile-time variation.
- Factory Method can appear as a step inside a template method.
- Bridge/State may look structurally similar but solve different concerns.
