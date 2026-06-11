# Builder Reference

## Intent
Builder separates the construction of a complex object from its representation so that the same construction process can create different representations.

## Problem Signal
- Constructor grows to ten or more parameters, most optional (telescoping constructor).
- Object initialization code is scattered across client call sites.
- Multiple representations of an object require nearly identical construction logic.
- Object graph construction is recursive or involves many nested objects.

## Solution Shape
1. Define a Builder interface declaring one method per configurable step.
2. Implement one Concrete Builder per product representation.
3. Each Concrete Builder accumulates state and exposes `build()` / `getProduct()`.
4. Optional: create a Director that encodes ordered step recipes.
5. Client instantiates a Concrete Builder, passes it to Director (or calls steps directly), then retrieves the product from the builder.

## Roles
- **Builder**: Interface declaring all construction step methods plus `reset()`.
- **Concrete Builder**: Implements Builder steps; assembles a specific product variant; owns `getProduct()` / `build()`.
- **Product**: The complex object being constructed. Different builders may produce unrelated product types.
- **Director**: Calls builder steps in a fixed order to produce a known configuration. Completely optional.
- **Client**: Creates Concrete Builder, optionally hands it to Director, fetches result from the builder.

## Applicability
Use when:
- Object construction requires many optional fields or nested configuration.
- Several representations of the same product share identical construction steps.
- Construction must be deferred, paused, or executed recursively (e.g., Composite trees).
- Encapsulating construction details from clients is desirable.

Avoid when:
- Product has only a few stable, required fields.
- Language already provides named/default parameters covering the same problem.
- Only one representation will ever exist and no director logic is needed.

## Refactor Recipe
1. Identify all parameters and initialization steps spread across constructors and client code.
2. Extract a Builder interface with one method per configurable step.
3. Create a Concrete Builder that assembles the existing product.
4. Move construction calls from client code to builder step methods.
5. Add `build()` / `getProduct()` and a `reset()` to the Concrete Builder.
6. If construction order matters or multiple clients share a recipe, introduce a Director.
7. Replace old constructor calls in client code with builder + director (or direct step calls).
8. Remove the bloated constructor once all call sites migrate.

## Validation Checklist
- Client code does not reference concrete product constructor directly.
- Director (if present) only calls methods on the Builder interface.
- Product is retrieved from the builder, not returned by Director.
- Adding a new representation requires only a new Concrete Builder.
- Builder can be reset and reused without leaking state from a previous build.

## Pros
- Eliminates telescoping constructors and optional-parameter sprawl.
- Same Director logic produces multiple product representations.
- Isolates complex construction code (Single Responsibility Principle).
- Allows step-by-step, deferred, or recursive construction.

## Cons
- Adds new classes (Builder interface, one Concrete Builder per variant, optional Director).
- Overkill for simple objects with few fields.
- `getProduct()` cannot be on the Builder interface when builders produce unrelated types.

## Relationship Notes
- Builder vs Abstract Factory: Abstract Factory returns a product family immediately; Builder assembles one complex product over multiple steps.
- Builder + Composite: Builder steps can call themselves recursively to build tree structures.
- Builder + Bridge: Director is the abstraction; Concrete Builders are the implementations.
- Abstract Factory / Builder / Prototype can all be implemented as Singletons.
- Projects often start with Factory Method and evolve to Builder as product complexity grows.
