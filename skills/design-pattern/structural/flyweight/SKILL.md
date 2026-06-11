---
name: flyweight
description: Teach and apply the Flyweight (Cache) pattern to reduce RAM usage by sharing immutable intrinsic state across massive numbers of similar objects. Use when users ask about Flyweight, object pooling, intrinsic vs extrinsic state, RAM-constrained object creation, or comparing Flyweight to Singleton or Prototype.
---

# Flyweight

Also known as: Cache

## Purpose
Help the user identify when RAM consumption from massive numbers of similar objects is the actual bottleneck, then design and apply Flyweight to extract shared immutable state into reusable objects while keeping unique state in lightweight context objects.

## When To Use
- User asks to explain the Flyweight pattern.
- The program creates a huge number of similar objects and RAM is measurably exhausted or close to it.
- Objects contain large chunks of state (textures, sprites, config blobs) that are identical across many instances.
- That shared state is read-only — it never changes after object creation.
- The number of unique intrinsic state combinations is far smaller than the number of object instances needed.

## Inputs
- The class driving high memory consumption and an estimate of instance count.
- A breakdown of fields: which are identical across many instances (intrinsic) vs. unique per instance (extrinsic).
- Whether a Flyweight Factory / pool is needed or clients can manage sharing manually.
- Concurrency requirements (shared flyweights accessed from multiple threads).
- Language and platform constraints (GC behavior, weak references, interning support).

## Workflow
1. Clarify the goal.
   Confirm whether RAM consumption from object count is the actual measured problem. Flyweight adds complexity — apply it only when profiling confirms the need.
2. Partition state.
   Classify every field as intrinsic (immutable, shared, identical across many objects) or extrinsic (unique per object or context, passed at call time).
3. Define the Flyweight class.
   Keep only intrinsic fields. Make them all immutable (set in constructor only, no setters, no public mutable fields). Move behavior that needs extrinsic state to methods that accept extrinsic data as parameters.
4. Create a Context class (if needed).
   Hold extrinsic state plus a reference to the flyweight. This is the lightweight object the system instantiates in bulk.
5. Implement the Flyweight Factory.
   Maintain a pool (map/cache) keyed by intrinsic state. Return an existing flyweight if one matches; create and cache a new one if not. Clients call the factory instead of the flyweight constructor directly.
6. Update client / container code.
   Replace direct object construction with factory calls. Store extrinsic state in context objects or the container collection.
7. Validate memory savings.
   Profile before and after. Confirm the number of flyweight instances equals the number of unique intrinsic state combinations, not the number of context objects.
8. Explain tradeoffs.
   Call out RAM savings vs. CPU cost of passing extrinsic state, added code complexity, and the immutability constraint on flyweights.

## Decision Branches
- If object count is large but intrinsic state variations are also large (most objects are unique):
  Flyweight provides no benefit — consider other memory strategies (streaming, paging, LOD).
- If shared state is mutable:
  Flyweight cannot be applied safely without synchronization. Either make the shared state immutable or use a different pattern.
- If only a handful of objects are created:
  Skip Flyweight entirely — the complexity is unjustified.
- If thread safety is required:
  Flyweight objects themselves are safe (immutable). The Flyweight Factory pool needs synchronization or a concurrent map.
- If context objects are also heavy:
  Profile and split further, or consider lazy loading / streaming the extrinsic state instead.

## Output Contract
When responding, provide:
1. A suitability verdict with a concrete estimate of memory impact (objects × bytes saved per object).
2. The intrinsic/extrinsic split mapped to the user's actual fields.
3. Flyweight class design (immutable fields, parameterized methods).
4. Flyweight Factory design (pool key, creation logic).
5. Context class or container update showing how extrinsic state is managed.
6. A minimal code sketch in the user's language.
7. Validation checklist.

## Quality Checks
- Flyweight class has no mutable state — constructor-only initialization, no setters.
- Flyweight methods accept extrinsic state as parameters, never store it.
- Flyweight Factory returns the same object reference for identical intrinsic state.
- Context objects hold extrinsic state plus a flyweight reference (not copies of intrinsic data).
- Client code does not call the flyweight constructor directly — all access via factory.
- Thread-safe factory when concurrent access is expected.
- Memory profile confirms meaningful RAM reduction after applying the pattern.

## Common Mistakes To Prevent
- Applying Flyweight without profiling — premature optimization that adds complexity for no measured gain.
- Including mutable or extrinsic state in the flyweight class (breaks sharing and correctness).
- Using object identity checks (`===`) on flyweights while assuming all contexts with the same intrinsic state are the same object — must go through the factory.
- Factory pool growing unbounded when intrinsic state has too many variations.
- Confusing Flyweight with Singleton: Singleton = one instance total; Flyweight = one instance per unique intrinsic state combination.

## References
- Detailed guide: [REFERENCE.md](./REFERENCE.md)
- Worked examples: [EXAMPLES.md](./EXAMPLES.md)
