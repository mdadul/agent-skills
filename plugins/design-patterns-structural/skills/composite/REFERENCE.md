# Composite Reference

## Intent
Composite represents part-whole hierarchies as trees so clients can treat individual objects (leaves) and groups (containers) uniformly.

## Problem Signal
- Domain has recursive containment (items containing items).
- Client code has repeated type checks for leaf vs container handling.
- Aggregation logic is duplicated at multiple nesting levels.
- New node types require widespread conditional updates.

## Solution Shape
1. Define a common component interface.
2. Implement leaf nodes for atomic behavior.
3. Implement container nodes that hold child components.
4. Delegate operations recursively from containers to children.
5. Aggregate child results and return unified outcomes.

## Roles
- Component: Shared API for all nodes.
- Leaf: Atomic node with no children.
- Composite/Container: Node with child components and recursive delegation.
- Client: Uses component abstraction uniformly.

## Applicability
Use when:
- The model is tree-structured and recursive operations are central.
- Uniform client handling across simple and complex nodes is desired.

Avoid when:
- Structure is a DAG/graph with shared ownership semantics.
- Flat collections solve the problem without recursive abstraction.

## Interface Design Notes
- Keep component interface minimal and domain-focused.
- Child-management methods can live:
  - only in container (clean ISP, less uniform construction), or
  - on component (uniform API, leaves may no-op/throw).

## Refactor Recipe
1. Identify branching hotspots where clients distinguish leaf/container.
2. Extract shared component operations.
3. Implement leaf classes using direct behavior.
4. Implement container with child collection and recursive delegation.
5. Move aggregation logic from clients into container methods.
6. Add invariants: cycle prevention, parent consistency, ordering rules.
7. Replace client conditionals with polymorphic component calls.

## Validation Checklist
- Recursive results match expected totals/renders for mixed-depth trees.
- Empty containers produce valid neutral behavior.
- No cycles can be introduced through child operations.
- Adding a new leaf/container type does not require client rewrites.
- Traversal performance is acceptable for expected depth/size.

## Pros
- Simplifies client code via polymorphism.
- Supports open-ended tree growth and new node types.
- Localizes recursive aggregation in container classes.

## Cons
- Hard to keep component interface minimal when node capabilities diverge.
- Deep trees can increase recursion and traversal costs.

## Relationship Notes
- Decorator has one child and augments behavior; Composite manages many children and aggregates.
- Visitor pairs well when adding operations frequently to stable tree types.
- Iterator can standardize traversal order over composite trees.
- Builder can simplify creation of complex composite trees.
