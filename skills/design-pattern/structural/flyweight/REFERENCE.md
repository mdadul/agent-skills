# Flyweight Reference

## Intent
Flyweight uses sharing to efficiently support a large number of fine-grained objects by separating immutable shared state (intrinsic) from unique per-instance state (extrinsic).

## Problem Signal
- A program instantiates millions of similar objects and runs out of RAM.
- Profiling shows most memory is consumed by duplicate data repeated across many object instances.
- Object fields can be cleanly split into a small set of shared constant values and a large set of per-instance varying values.

## Core Concepts

### Intrinsic State
- Immutable data stored inside the flyweight.
- Identical across many objects (e.g., tree species name, texture, sprite, color).
- Set once in the constructor; no setters or mutable public fields.
- Defines the identity of the flyweight — used as the pool key.

### Extrinsic State
- Unique per context or changes over time (e.g., x/y coordinates, velocity, health).
- Never stored in the flyweight.
- Passed as parameters to flyweight methods at call time.
- Stored in Context objects or the container/collection managing all instances.

## Roles
- **Flyweight**: Stores intrinsic state only. Methods that need extrinsic state accept it as parameters. Must be immutable.
- **Flyweight Factory**: Maintains a pool (map) keyed by intrinsic state. Returns an existing flyweight or creates and caches a new one. Clients always go through the factory.
- **Context**: Stores extrinsic state plus a reference to a flyweight. Represents a single "logical" object. Lightweight — can be created in the millions.
- **Container / Client**: Holds the collection of contexts. Calls factory to obtain flyweights. Passes extrinsic state to flyweight methods.

## Applicability
Use when:
- The application creates a huge number of objects and RAM is the bottleneck.
- Most object state can be made extrinsic (moved outside the object).
- The number of unique intrinsic state combinations is much smaller than the total object count.
- Object identity is not required (flyweights are value-like, not entity-like).

Avoid when:
- Object count is small — added complexity is not worth the savings.
- Intrinsic state variations are nearly as numerous as total object count (no sharing possible).
- Shared state must be mutable (breaks the pattern's safety guarantee).
- The extrinsic state itself is large or expensive to recompute each call.

## Refactor Recipe
1. Profile the application to confirm RAM exhaustion from a specific class.
2. List all fields of that class. Classify each as intrinsic or extrinsic.
3. Create a new Flyweight class containing only intrinsic fields with constructor initialization and no setters.
4. Move methods that use extrinsic fields to accept those values as parameters instead.
5. Create a Context class holding extrinsic fields + a `flyweight: Flyweight` reference.
6. Implement a Flyweight Factory with a `Map<intrinsicKey, Flyweight>` pool.
7. Replace all `new OriginalClass()` sites with factory calls + context creation.
8. Delete or repurpose the original class if no longer needed.

## Memory Impact Estimation
```
Before: N objects × (intrinsic_bytes + extrinsic_bytes)
After:  K flyweights × intrinsic_bytes  +  N contexts × (extrinsic_bytes + pointer_size)

Savings ≈ (N - K) × intrinsic_bytes  (meaningful when N >> K and intrinsic_bytes is large)
```
Calculate this before committing to the refactor.

## Validation Checklist
- Flyweight class has no mutable fields.
- Flyweight methods never store extrinsic state as instance fields.
- Factory returns identical object references for identical intrinsic keys.
- Context objects are small: extrinsic fields + one flyweight reference.
- Total flyweight pool size equals the number of unique intrinsic state combinations.
- RAM profile shows meaningful improvement after the change.

## Pros
- Dramatic RAM reduction when many objects share large chunks of identical state.
- Flyweights are inherently thread-safe (immutable).

## Cons
- CPU cost: extrinsic state must be passed on every method call or looked up in context.
- Code complexity increases: the original class is split into three (Flyweight, Context, Factory).
- Harder to debug — logical object state is spread across two physical objects.
- Not applicable if shared state is mutable.

## Relationship Notes
- **Flyweight vs Singleton**: Singleton = one instance total, may be mutable. Flyweight = one instance per unique intrinsic state combination, must be immutable. A Singleton is a degenerate Flyweight where all intrinsic states collapse to one.
- **Flyweight + Composite**: Shared leaf nodes in a Composite tree can be Flyweights to save RAM on repeated leaves.
- **Flyweight vs Facade**: Flyweight creates many tiny shared objects; Facade creates one object representing a whole subsystem.
- **Flyweight + Prototype**: In Composite/Flyweight-heavy systems, Prototype can clone assembled structures rather than rebuilding them.
- **Flyweight vs Object Pool**: Object Pool reuses mutable objects to avoid GC pressure (returned and recycled). Flyweight shares immutable objects to reduce duplication (never recycled, just shared).
