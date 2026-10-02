---
name: bridge
description: Teach and apply the Bridge pattern to decouple abstraction from implementation across two independent class hierarchies. Use when users ask about Bridge, class hierarchy explosion, orthogonal dimensions, platform-independent abstractions, or comparing Bridge to Adapter or Strategy.
---

# Bridge

## Purpose
Help the user identify when inheritance is being used to extend a class in two independent directions, then design and apply Bridge to replace that hierarchy explosion with composition.

## When To Use
- User asks to explain the Bridge pattern.
- A class hierarchy is growing exponentially because two independent dimensions are mixed (e.g., shape × color, GUI × OS API, notification × channel).
- A class needs to work with multiple implementations that may vary or be swapped at runtime.
- User wants to extend abstraction and implementation independently without touching the other side.
- Platform-specific code needs to be isolated from high-level control logic.

## Inputs
- The class or hierarchy exhibiting combinatorial growth.
- The two (or more) independent dimensions driving that growth.
- Whether implementations need to be swappable at runtime.
- Language and framework constraints (interfaces, multiple inheritance support, DI patterns).
- Existing vs. greenfield code (greenfield → design up-front; existing → incremental extraction).

## Workflow
1. Clarify the goal.
   Confirm whether the user wants explanation, code review, or implementation/refactor.
2. Identify orthogonal dimensions.
   Find the two independent axes of variation causing the class explosion. Name them clearly (abstraction / implementation, GUI / platform, remote / device, etc.).
3. Define the Implementation interface.
   Extract the low-level primitive operations needed by the abstraction into a common interface. These are the operations all concrete implementations must provide.
4. Create Concrete Implementations.
   One class per platform/variant implementing the Implementation interface.
5. Define the Abstraction.
   Keep the high-level control logic here. Replace direct platform code with calls to the implementation object via the interface. Add a reference field holding the Implementation.
6. Create Refined Abstractions (if needed).
   Subclass the Abstraction for variant control logic without touching implementations.
7. Wire in the Client.
   Client constructs a concrete implementation, injects it into the abstraction's constructor, and then works only with the abstraction.
8. Validate behavior.
   Confirm a new implementation can be added without changing any abstraction class, and a new abstraction variant can be added without changing any implementation class.
9. Explain tradeoffs.
   Call out added indirection and up-front design cost vs. reduction in class count and independent extensibility.

## Decision Branches
- If only one dimension is varying:
  Avoid Bridge; a simple interface + implementation pair (or Strategy) is sufficient.
- If adapting an existing incompatible interface:
  Prefer Adapter over Bridge; Bridge is designed up-front, Adapter is applied after the fact.
- If behavior needs to be swapped at runtime per call (not per object):
  Consider Strategy instead; Bridge and Strategy are structurally similar but Bridge focuses on hierarchy decoupling, Strategy on algorithm interchangeability.
- If implementations need to be selected based on abstraction families:
  Combine with Abstract Factory to encapsulate pairing logic.
- If construction of combined abstraction+implementation is complex:
  Combine with Builder (Director acts as abstraction, builders act as implementations).

## Output Contract
When responding, provide:
1. A concise suitability verdict (why Bridge or why not).
2. Named roles mapped to the user's domain (which class is Abstraction, which is Implementation).
3. Step-by-step migration plan from current inheritance hierarchy to Bridge composition.
4. A minimal code sketch in the user's language.
5. Validation checklist confirming independent extensibility of both hierarchies.

## Quality Checks
- Abstraction class holds a reference to the Implementation interface, not any concrete class.
- Abstraction methods delegate low-level work to the implementation; high-level logic stays in the abstraction.
- Adding a new concrete implementation requires zero changes to any abstraction class.
- Adding a new refined abstraction requires zero changes to any implementation class.
- Client code depends only on the Abstraction and Implementation interface, not concrete classes.
- Implementation can be swapped at runtime if that is a requirement.

## Common Mistakes To Prevent
- Confusing Bridge with Adapter: Bridge is a deliberate upfront design; Adapter retrofits incompatible interfaces.
- Confusing Bridge with Strategy: they look structurally identical but solve different problems — communicate the distinction clearly.
- Leaving direct references to concrete implementations inside the abstraction.
- Creating a Bridge for a class with only one dimension of variation (over-engineering).
- Putting business logic in the Implementation layer instead of keeping it in the Abstraction.

## References
- Detailed guide: [REFERENCE.md](./REFERENCE.md)
- Worked examples: [EXAMPLES.md](./EXAMPLES.md)
