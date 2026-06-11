---
name: prototype
description: Teach and apply the Prototype pattern to copy existing objects through a cloning interface without coupling client code to concrete classes. Use when users ask about Prototype or Clone, replacing repeated initialization with preconfigured instances, handling copy semantics, or comparing Prototype with Factory Method, Builder, or Memento.
---

# Prototype

Also known as: Clone

## Purpose
Help the user decide whether Prototype is appropriate, then design and apply safe cloning workflows, including deep-copy boundaries and optional prototype registries.

## When To Use
- User asks to explain Prototype or Clone pattern.
- Client code must copy objects but should not depend on their concrete classes.
- Initialization logic is expensive or repetitive and preconfigured instances would reduce duplication.
- System needs reusable presets rather than many configuration-only subclasses.
- Code works with third-party/interface-based objects where concrete types are unknown.

## Inputs
- Object graph and copy requirements (exact copy, partial copy, reset fields).
- Field-level semantics (immutable, mutable, shared, external resource handles).
- Expected clone depth (shallow vs deep) and circular reference behavior.
- Whether a prototype registry is needed for preset lookup.
- Language constraints for constructors, copy methods, and serialization.

## Workflow
1. Clarify the goal.
   Confirm whether the user needs conceptual guidance, code review, or implementation/refactor.
2. Check applicability.
   Use Prototype when copy behavior should be polymorphic and independent from concrete classes.
3. Define clone contract.
   Specify what clone means for this domain: deep/shallow boundaries, shared references, and fields to reinitialize.
4. Introduce prototype interface.
   Add a cloning method (`clone`, `copy`, or equivalent) to relevant abstractions.
5. Implement concrete clone logic.
   Ensure each class copies inherited and own fields correctly, preserving invariants.
6. Handle graph complexity.
   Address linked objects, cycles, or identity-sensitive references explicitly.
7. Add registry if needed.
   Store prebuilt prototypes and return cloned instances by key/criteria.
8. Replace manual duplication paths.
   Swap ad hoc constructor+setter copy code with polymorphic clone/registry calls.
9. Validate copy correctness.
   Verify clone independence where required, intentional sharing where allowed, and invariant safety.

## Decision Branches
- If only a few scalar fields are copied and constructors are simple:
  Prefer direct construction or static factories.
- If object assembly is step-based and not copy-based:
  Prefer Builder.
- If subclassing is already stable and variations are creation-type driven:
  Prefer Factory Method.
- If clones involve many external resources or non-copyable handles:
  Prefer explicit rehydration/factory workflows over blind cloning.
- If repeated presets are needed:
  Add a prototype registry to centralize configured templates.

## Output Contract
When responding, provide:
1. A suitability verdict (use Prototype or choose an alternative).
2. Clone policy summary (deep/shallow boundaries and shared-reference rules).
3. Role mapping (Prototype interface, Concrete Prototypes, optional Registry, Client usage).
4. Incremental migration plan from manual copying to clone-based flow.
5. Minimal code sketch in the user's language, including one tricky field/graph case.
6. Validation checklist proving correctness and independence guarantees.

## Quality Checks
- Clone API is available at the abstraction level used by the client.
- Every concrete type returns its own type-compatible clone behavior.
- Inheritance chains copy parent and child state correctly.
- Mutable nested objects are deep-cloned when independence is required.
- External resources are re-bound, shared intentionally, or excluded by policy.
- Tests verify original and clone divergence after mutation.

## Common Mistakes To Prevent
- Copying only top-level fields while leaking shared mutable internals.
- Forgetting to copy parent/private state in inheritance hierarchies.
- Using serialization-based deep copy without honoring domain invariants.
- Treating all references as deep-copied when some must stay shared.
- Returning partially initialized clones.

## References
- Detailed guide: [REFERENCE.md](./REFERENCE.md)
- Worked examples: [EXAMPLES.md](./EXAMPLES.md)
