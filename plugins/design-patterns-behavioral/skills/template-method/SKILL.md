---
name: template-method
description: Teach and apply the Template Method pattern to define an algorithm skeleton in a base class while allowing subclasses to override selected steps without changing overall flow. Use when users ask about Template Method, extracting duplicate algorithm structure, superclass hooks, final template flows, or comparing Template Method with Strategy or Factory Method.
---

# Template Method

## Purpose
Help the user decide whether Template Method is appropriate, then design and implement stable algorithm skeletons with controlled extension points for subclasses.

## When To Use
- User asks to explain Template Method pattern.
- Multiple classes share the same high-level algorithm flow with minor step differences.
- Code duplication exists across subclasses or sibling classes for orchestration logic.
- Team needs to preserve algorithm order while allowing variation in certain steps.
- Runtime strategy swapping is not required and inheritance-based variation is acceptable.

## Inputs
- Current algorithm flow and step boundaries.
- Common vs varying steps across implementations.
- Required extension points (abstract steps, default steps, hooks).
- Inheritance constraints and subclass count.
- Contract rules/invariants that must remain fixed.

## Workflow
1. Clarify the goal.
   Confirm whether the user needs conceptual guidance, architecture review, or implementation/refactor.
2. Check applicability.
   Use Template Method when algorithm shape is stable and only step implementations vary.
3. Identify algorithm skeleton.
   Break the process into ordered steps and mark fixed sequencing constraints.
4. Design base class.
   Implement template method orchestrating step calls; make it non-overridable where language supports it.
5. Classify step types.
   Mark steps as abstract (must override), default (may override), or hooks (optional empty extension points).
6. Extract shared logic.
   Move duplicated common behavior into base class default steps.
7. Implement concrete subclasses.
   Override required steps and optional customizations while preserving base invariants.
8. Refactor clients.
   Replace conditional class-specific orchestration with polymorphic calls to template method.
9. Validate behavior.
   Ensure algorithm order is preserved across subclasses and variations remain local to steps.

## Decision Branches
- If behavior must switch dynamically at runtime per object/request:
  Prefer Strategy.
- If you need to package operations as objects for queueing/undo/history:
  Prefer Command.
- If only object creation varies inside a broader algorithm skeleton:
  Consider Factory Method as a template step.
- If subclassing burden becomes high or inheritance is constrained:
  Prefer composition-based alternatives.
- If template method has too many steps and grows fragile:
  Split into smaller templates or extract subordinate collaborators.

## Output Contract
When responding, provide:
1. A suitability verdict (Template Method or better alternative).
2. Skeleton summary (ordered steps, invariants, extension points).
3. Role mapping (Abstract Base, Template Method, Concrete Subclasses, Client).
4. Incremental migration plan from duplicated algorithms to template base class.
5. Minimal code sketch in the user's language showing abstract/default/hook steps.
6. Validation checklist proving order stability and safe extensibility.

## Quality Checks
- Template method defines algorithm order once in base class.
- Subclasses cannot bypass required invariants enforced by the template flow.
- Abstract/default/hook boundaries are explicit and documented.
- Shared logic is centralized; subclass duplication is minimized.
- Adding a new variant requires subclass implementation, not base flow edits.
- Tests verify both base invariants and subclass-specific step behavior.

## Common Mistakes To Prevent
- Letting subclasses override template flow and break ordering guarantees.
- Making every step abstract and losing shared behavior benefits.
- Overusing hooks and creating unclear extension semantics.
- Forcing inheritance where composition would be simpler.
- Building giant template methods that are hard to evolve safely.

## References
- Detailed guide: [REFERENCE.md](./REFERENCE.md)
- Worked examples: [EXAMPLES.md](./EXAMPLES.md)
