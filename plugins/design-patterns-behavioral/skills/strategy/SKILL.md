---
name: strategy
description: Teach and apply the Strategy pattern to isolate interchangeable algorithms behind a common interface and swap them at runtime without changing context logic. Use when users ask about Strategy, algorithm selection, replacing large conditionals, runtime behavior switching, or comparing Strategy with State, Command, or Template Method.
---

# Strategy

## Purpose
Help the user decide whether Strategy is appropriate, then design and implement interchangeable algorithm objects that keep context code stable and easy to extend.

## When To Use
- User asks to explain Strategy pattern.
- A context class contains large conditionals selecting algorithm variants.
- Multiple classes differ only by one behavior and duplicate surrounding logic.
- Runtime algorithm switching is required based on user input or environment.
- Team wants to add new algorithm variants without editing core context class.

## Inputs
- Context responsibility and algorithm variation points.
- Candidate algorithm variants and selection criteria.
- Runtime switching needs (static config vs dynamic per request/user).
- Interface contract for algorithm input/output and error handling.
- Performance constraints and statefulness requirements.

## Workflow
1. Clarify the goal.
   Confirm whether the user needs conceptual guidance, architecture review, or implementation/refactor.
2. Check applicability.
   Use Strategy when behaviors are true algorithm variants, not workflow steps or command history operations.
3. Define strategy contract.
   Create one interface describing the algorithm operation expected by context.
4. Extract concrete strategies.
   Move each algorithm variant into its own implementation.
5. Refactor context.
   Replace conditional logic with delegation to current strategy via interface.
6. Add strategy selection.
   Choose strategy in client/config layer, or expose safe runtime setter in context.
7. Stabilize boundaries.
   Keep context focused on orchestration and strategy focused on algorithm details.
8. Optimize composition.
   Use dependency injection/factory if strategy construction is non-trivial.
9. Validate behavior.
   Ensure all strategies satisfy same contract and context behavior remains correct when swapping.

## Decision Branches
- If behavior changes because of internal lifecycle/state transitions:
  Prefer State.
- If operation needs queueing, undo/redo, or command history:
  Prefer Command.
- If algorithm skeleton is fixed with overridable steps at compile-time:
  Prefer Template Method.
- If only two stable variants exist and change is unlikely:
  Prefer direct conditional for simplicity.
- If strategies require drastically different inputs/outputs:
  Revisit abstraction; Strategy may be forcing an unnatural interface.

## Output Contract
When responding, provide:
1. A suitability verdict (Strategy or better alternative).
2. Variation-point summary (what algorithm concern is being extracted).
3. Role mapping (Context, Strategy Interface, Concrete Strategies, Client selection).
4. Incremental migration plan from conditional logic to strategy delegation.
5. Minimal code sketch in the user's language showing runtime strategy swap.
6. Validation checklist proving interchangeability and behavior parity.

## Quality Checks
- Context depends only on strategy interface, not concrete strategy classes.
- Each strategy implements the same contract and returns compatible results.
- Client/config layer owns strategy selection policy.
- Context contains no leaked algorithm-specific branching after refactor.
- Adding a new strategy requires no context changes.
- Strategy-specific state is isolated and thread-safe as required.

## Common Mistakes To Prevent
- Keeping large `if/switch` blocks in context after introducing strategies.
- Letting strategies depend on context internals too tightly.
- Mixing workflow orchestration responsibilities into strategies.
- Using Strategy where State or Command better matches intent.
- Over-engineering when algorithm variants are few and stable.

## References
- Detailed guide: [REFERENCE.md](./REFERENCE.md)
- Worked examples: [EXAMPLES.md](./EXAMPLES.md)
