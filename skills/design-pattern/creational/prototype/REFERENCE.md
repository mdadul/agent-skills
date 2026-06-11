# Prototype Reference

## Intent
Prototype creates new objects by cloning existing ones, enabling copy operations without coupling clients to concrete classes.

## Problem Signal
- Manual copy logic is duplicated and fragile.
- Client code must know concrete classes only to duplicate objects.
- Repeated initialization creates many configuration-only subclasses.
- Copying is inconsistent for nested mutable data.

## Solution Shape
1. Define a prototype interface with a clone operation.
2. Implement concrete clone logic per class.
3. Copy parent and child state safely across inheritance boundaries.
4. Specify deep vs shallow behavior for each field group.
5. Optionally add a prototype registry for reusable presets.

## Roles
- Prototype: Declares cloning operation.
- Concrete Prototype: Implements class-specific copy behavior.
- Client: Requests clones via abstraction.
- Prototype Registry (optional): Catalog of named/keyed prototypes.

## Applicability
Use when:
- Objects must be copied polymorphically through interfaces.
- Common configurations should be reused as templates.
- Construction is expensive and cloning is cheaper/safer.

Avoid when:
- Object graphs include complex non-copyable resources without clear policies.
- Direct construction is simple and copy behavior is trivial.

## Clone Policy Checklist
- Which fields are deep-cloned?
- Which fields are intentionally shared?
- Which fields are reset/re-generated in clone?
- How are circular references handled?
- How are external handles (db sockets/files) treated?

## Refactor Recipe
1. Identify copy hotspots and define clone semantics.
2. Add clone operation to abstract type used by clients.
3. Implement concrete clone per type, including inherited fields.
4. Add tests for identity separation and invariant preservation.
5. Introduce registry only when preset reuse is frequent.
6. Replace manual copy paths with clone/registry calls.

## Validation Checklist
- Cloned objects satisfy same invariants as constructed objects.
- Mutation in clone does not leak to original unless policy says shared.
- Circular references are copied/relinked without recursion failures.
- Registry returns fresh clones, never shared mutable prototypes.

## Pros
- Reduces coupling to concrete classes.
- Reuses preconfigured objects and cuts initialization duplication.
- Works well with dynamic or third-party object families.

## Cons
- Deep copying complex graphs is error-prone.
- Copy semantics must be carefully documented and tested.

## Relationship Notes
- Alternative to inheritance-heavy configuration hierarchies.
- Can complement Abstract Factory by using prototypes as creation source.
- Sometimes simpler than Memento for straightforward state snapshots.
