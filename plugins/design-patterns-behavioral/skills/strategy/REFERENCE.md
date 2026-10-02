# Strategy Reference

## Intent
Strategy encapsulates interchangeable algorithms behind a shared interface so context behavior can vary without modifying context code.

## Problem Signal
- One class keeps growing with `if/switch` algorithm branches.
- Similar classes duplicate surrounding orchestration but differ in one calculation.
- New algorithm variants frequently cause merge conflicts in the same context file.
- Runtime behavior toggles are brittle and hard to test.

## Solution Shape
1. Define strategy interface for algorithm operation.
2. Implement one concrete strategy per variant.
3. Keep context holding a strategy reference.
4. Delegate algorithm execution from context to selected strategy.
5. Move selection policy to client/config layer.

## Roles
- Context: Uses strategy through interface and owns orchestration.
- Strategy Interface: Common algorithm contract.
- Concrete Strategy: One algorithm implementation.
- Client: Chooses and injects strategy.

## Applicability
Use when:
- Multiple algorithm variants share inputs/outputs and are swappable.
- Runtime or configuration-driven algorithm selection is required.
- You want to add new variants without touching context internals.

Avoid when:
- Variants are few, stable, and unlikely to change.
- Behavior differences are lifecycle state transitions (State is better).

## Refactor Recipe
1. Identify conditional branch region in context.
2. Extract shared algorithm contract.
3. Implement concrete strategies incrementally (one branch at a time).
4. Replace branch with strategy delegation.
5. Move selection into client/factory/config.
6. Add tests for each strategy plus context-swap scenarios.

## Validation Checklist
- Context compiles without concrete strategy imports where possible.
- Swapping strategy changes algorithm behavior only, not orchestration semantics.
- New strategy can be introduced without context edits.
- Contract invariants (errors, units, ordering) hold across all strategies.
- Selection policy is explicit and testable.

## Pros
- Reduces conditional complexity in context.
- Encourages Open/Closed extension for new algorithms.
- Improves isolated testing of algorithm variants.

## Cons
- Adds more classes/objects.
- Clients must understand strategy differences to choose correctly.

## Relationship Notes
- State shares structure with Strategy but models internal state transitions.
- Command encapsulates executable requests with history/queue concerns.
- Template Method varies algorithm steps via inheritance, not runtime composition.
- Bridge separates abstraction/implementation dimensions, not algorithm variants in one context.
