---
name: builder
description: Teach and apply the Builder pattern to construct complex objects step by step, separating construction from representation. Use when users ask about Builder, telescoping constructors, step-by-step object assembly, Director/Builder roles, or comparing Builder to Factory patterns.
---

# Builder

Also known as: Step Builder, Fluent Builder

## Purpose
Help the user decide whether Builder is appropriate, then design and apply it to replace complex constructors or scattered initialization code with a clean, step-by-step construction API.

## When To Use
- User asks to explain the Builder pattern.
- Constructor has many parameters, most of which are optional (telescoping constructor smell).
- The same construction steps need to produce different representations of an object.
- Object construction involves many configuration steps that must occur in a defined order.
- User wants to build complex Composite trees or nested object graphs step by step.
- User needs to defer or skip certain construction steps depending on the target object type.

## Inputs
- The complex object being constructed (the Product) and its required/optional fields.
- Number of representations or variants needed (drives need for multiple Concrete Builders).
- Whether a fixed construction order exists (drives need for a Director).
- Constraints: language, framework style, immutability requirements, test needs.

## Workflow
1. Clarify the goal.
   Confirm whether the user wants explanation, code review, or implementation/refactor.
2. Check applicability.
   Use Builder when products are complex (many fields, nested objects, or multiple representations). Avoid it for simple objects with two or three stable fields.
3. Define roles.
   Identify: Product(s), Builder interface, Concrete Builder(s), Director (optional), and Client.
4. Extract construction steps.
   Define step methods on the Builder interface covering all configurable parts of the product.
5. Implement Concrete Builders.
   Each builder implements all steps and provides its own `getProduct()` / `build()` method.
6. Introduce Director (if needed).
   Encapsulate common construction recipes (e.g., `constructSportsCar`, `constructSUV`) in a Director class that calls builder steps in order.
7. Wire the Client.
   Client creates a concrete builder, passes it to Director (or calls steps directly), then retrieves the product from the builder.
8. Validate behavior.
   Confirm product state matches intended configuration; confirm adding a new representation requires only a new Concrete Builder, not changes to existing code.
9. Explain tradeoffs.
   Call out added class count vs readability and flexibility gained.

## Decision Branches
- If object has only a few stable fields with no optional variations:
  Prefer a plain constructor or static factory; avoid Builder overhead.
- If multiple related product families are needed (not just representations of one product):
  Consider Abstract Factory instead of or alongside Builder.
- If Director is not needed (only one construction sequence):
  Client may call builder steps directly; Director is optional.
- If the language supports named/default parameters (Kotlin, Python, Swift):
  Evaluate whether Builder adds value over a data class with defaults before introducing it.
- If products from different builders must share a common interface:
  Declare a common Product interface; otherwise leave products unrelated.

## Output Contract
When responding, provide:
1. A concise suitability verdict (why Builder or why not).
2. A mapped design with all roles (Builder interface, Concrete Builder(s), Director, Product(s), Client).
3. Step-by-step migration plan from current construction code to Builder.
4. A minimal code sketch in the user's language.
5. Validation checklist proving behavior parity and extension ease.

## Quality Checks
- Builder interface covers all configurable construction steps.
- Director (if present) does not depend on concrete builder or product classes.
- Client retrieves the product from the builder, not the director.
- Adding a new product representation requires only a new Concrete Builder, not changes to Director or Client.
- `reset()` / `build()` returns a fresh product so builder instances can be reused.
- Tests cover at least one construction path per Concrete Builder.

## Common Mistakes To Prevent
- Having the Director return the product (couples Director to a concrete type).
- Declaring `getProduct()` on the Builder interface when builders produce unrelated types.
- Forgetting to reset builder state after `build()` / `getProduct()`.
- Applying Builder to simple objects that just need named constructor parameters.
- Letting client code call steps in an arbitrary order that leaves the product in an inconsistent state.

## References
- Detailed guide: [REFERENCE.md](./REFERENCE.md)
- Worked examples: [EXAMPLES.md](./EXAMPLES.md)
