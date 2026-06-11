# Bridge Reference

## Intent
Bridge decouples an abstraction from its implementation so that the two can vary independently.

## Problem Signal
- A class hierarchy is growing exponentially because two independent concepts (e.g., shape × color, GUI × OS, remote × device) are encoded together through inheritance.
- Every new variant in one dimension requires creating new subclasses for every variant in the other dimension.
- High-level control logic and low-level platform code are tangled in the same class.

## Solution Shape
1. Identify the two orthogonal dimensions.
2. Extract the lower-level dimension into an Implementation interface plus Concrete Implementations.
3. Keep the higher-level dimension as an Abstraction class that holds a reference to an Implementation object.
4. Abstraction delegates all low-level work to the Implementation via the interface.
5. Client links a concrete implementation to the abstraction at construction time.

## Roles
- **Abstraction**: High-level control layer. Holds a `protected` reference to an Implementation object. Defines the interface the client uses.
- **Refined Abstraction**: Subclass of Abstraction adding variant control logic without affecting implementations.
- **Implementation** (interface): Declares primitive operations required by the Abstraction. May be entirely different from the Abstraction's interface.
- **Concrete Implementation**: Platform- or variant-specific implementation of the Implementation interface.
- **Client**: Instantiates a Concrete Implementation, injects it into an Abstraction, then works exclusively via the Abstraction.

## Applicability
Use when:
- A class grows in two independent dimensions and inheritance would cause a combinatorial explosion.
- Implementation details should be hidden from the client and replaceable at runtime.
- Both abstraction and implementation need to be extended independently.
- A monolithic class mixes high-level orchestration with low-level platform code.

Avoid when:
- Only one dimension varies (a simple interface + Strategy is lighter).
- You are retrofitting incompatible interfaces (use Adapter instead).
- The coupling between abstraction and implementation is inherently tight and unlikely to change.

## Refactor Recipe
1. Find the class with hierarchy explosion; name the two independent dimensions.
2. Define an Implementation interface covering the low-level primitive operations.
3. Create one Concrete Implementation per platform/variant.
4. Refactor the original class into an Abstraction that references an Implementation object.
5. Replace all internal platform-specific calls with calls to `this.implementation.*`.
6. If variant high-level logic exists, create Refined Abstractions by subclassing Abstraction.
7. Update client code to inject a Concrete Implementation into the Abstraction constructor.
8. Delete the now-redundant combined subclasses.

## Validation Checklist
- `Abstraction` stores an `Implementation` reference, never a concrete class reference.
- Adding a new Concrete Implementation touches zero Abstraction classes.
- Adding a new Refined Abstraction touches zero Implementation classes.
- Client depends only on `Abstraction` and `Implementation` interfaces.
- Low-level operations live exclusively in Concrete Implementations.
- High-level logic lives exclusively in Abstraction / Refined Abstractions.

## Pros
- Eliminates combinatorial class explosion.
- Open/Closed Principle: both hierarchies open for extension, closed for modification.
- Single Responsibility Principle: high-level logic in abstraction, platform details in implementation.
- Implementations are swappable at runtime.
- Platform-independent client code.

## Cons
- Adds indirection; code is harder to trace for simple scenarios.
- Requires up-front identification of orthogonal dimensions — misapplied, it over-engineers cohesive classes.

## Relationship Notes
- **Bridge vs Adapter**: Bridge is designed upfront to allow independent variation; Adapter is applied after the fact to reconcile incompatible interfaces.
- **Bridge vs Strategy**: Structurally nearly identical (both use composition), but Bridge addresses hierarchy decomposition across two dimensions while Strategy addresses interchangeable algorithms on one dimension. Intent and context differ.
- **Bridge + Abstract Factory**: Abstract Factory can manage which Concrete Implementation is paired with which Abstraction family, hiding that complexity from the client.
- **Bridge + Builder**: Director plays the Abstraction role; Concrete Builders act as Implementations.
- **State, Strategy, Adapter**: All are composition-based patterns with similar structure but different intents — call out the distinction when comparing.
