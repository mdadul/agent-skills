---
name: composite
description: Teach and apply the Composite pattern to model tree structures where leaves and containers share a common interface and can be handled uniformly. Use when users ask about Composite or Object Tree, recursive aggregation, uniform leaf/container operations, or comparing Composite with Decorator, Visitor, or Iterator.
---

# Composite

Also known as: Object Tree

## Purpose
Help the user decide whether Composite is appropriate, then design and implement tree-shaped object models that let clients treat simple and complex nodes uniformly.

## When To Use
- User asks to explain Composite or Object Tree pattern.
- Domain can be modeled naturally as a recursive tree (nodes with child nodes).
- Client should invoke the same operations on both leaves and containers.
- Logic requires recursive aggregation (price totals, rendering, permissions, metrics, traversal).
- Feature growth needs extensible node types without rewriting client code.

## Inputs
- Core domain entities and parent/child relationships.
- Candidate component operations shared by leaves and containers.
- Aggregation semantics (sum, merge, short-circuit, fold).
- Tree constraints (acyclic, max depth, ordering, identity, mutability rules).
- Runtime concerns (performance, traversal strategy, concurrency, serialization).

## Workflow
1. Clarify the goal.
   Confirm whether the user needs conceptual guidance, architecture review, or implementation/refactor.
2. Check applicability.
   Use Composite when the model is genuinely hierarchical and clients need uniform node handling.
3. Define component contract.
   Extract operations meaningful for both leaves and containers.
4. Model leaves.
   Implement concrete leaf classes that perform actual domain work directly.
5. Model containers.
   Implement composite/container classes that store child components and delegate recursively.
6. Add child management boundaries.
   Define add/remove/list behavior in container (and optionally on component with no-op/guard in leaves).
7. Refactor client usage.
   Replace type checks and special-case branching with component-polymorphic calls.
8. Protect tree integrity.
   Enforce no cycles, valid parent links, and domain invariants.
9. Validate recursive behavior.
   Verify correctness for deep nesting, empty containers, and mixed node types.

## Decision Branches
- If structure is not tree-like or requires graph semantics with shared parents:
  Prefer explicit graph model over Composite.
- If only one wrapped child is needed to extend behavior:
  Prefer Decorator.
- If operations frequently change but node structure is stable:
  Consider Visitor for operation extension.
- If traversal behavior needs abstraction from structure:
  Consider Iterator with or without Composite.
- If child-management methods on leaf violate interface clarity for your domain:
  Keep child operations only on container and accept non-uniform construction API.

## Output Contract
When responding, provide:
1. A suitability verdict (Composite or better alternative).
2. Tree model summary (component, leaf types, container types, child rules).
3. Role mapping (Client, Component, Leaf, Container/Composite).
4. Incremental migration plan from branching/type-check code to recursive polymorphism.
5. Minimal code sketch in the user's language showing one leaf and one container operation.
6. Validation checklist proving recursive correctness and tree integrity.

## Quality Checks
- Component interface includes only truly shared operations.
- Containers delegate to children and aggregate deterministically.
- Leaves do not depend on container internals.
- Tree invariants prevent cycles and invalid parent-child references.
- Client code works through component abstraction, not concrete type checks.
- Deeply nested and empty-tree edge cases are covered by tests.

## Common Mistakes To Prevent
- Forcing unrelated operations into component interface.
- Mixing business aggregation logic into clients instead of container recursion.
- Allowing cyclic references that cause infinite recursion.
- Overusing Composite for flat data where simple collections are enough.
- Coupling container behavior to specific leaf subclasses.

## References
- Detailed guide: [REFERENCE.md](./REFERENCE.md)
- Worked examples: [EXAMPLES.md](./EXAMPLES.md)
