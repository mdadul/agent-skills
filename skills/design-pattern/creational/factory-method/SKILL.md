---
name: factory-method
description: Teach and apply the Factory Method pattern to remove direct constructor coupling and enable extensible object creation via creator subclasses. Use when users ask about Factory Method, virtual constructor, replacing new calls, introducing creator/product hierarchies, or comparing Factory Method to Abstract Factory.
---

# Factory Method

Also known as: Virtual Constructor

## Purpose
Help the user decide whether Factory Method is appropriate, then design and apply it with safe, incremental refactoring.

## When To Use
- User asks to explain Factory Method or Virtual Constructor.
- Code has direct `new` calls tightly coupled to concrete classes.
- User needs to add new product types without changing client logic repeatedly.
- User wants to provide framework/library extension points through inheritance.
- User wants creation that may return cached/pooled instances instead of always creating new ones.

## Inputs
- Current construction hotspots (where concrete constructors are called).
- Target product family and shared product interface.
- Expected variants (existing and near-term new product types).
- Constraints: language, framework style, inheritance limits, test requirements.

## Workflow
1. Clarify the goal.
   Confirm whether the user wants explanation, code review, or implementation/refactor.
2. Check applicability.
   Use Factory Method when creation varies by subclass and client code should stay product-agnostic.
3. Define roles.
   Identify Product interface, Concrete Products, Creator, and Concrete Creators.
4. Refactor incrementally.
   Replace direct constructor usage in creator business logic with a factory method call.
5. Split variants.
   Move per-product construction into creator subclasses by overriding the factory method.
6. Preserve client contract.
   Ensure client code depends only on product abstractions.
7. Validate behavior.
   Verify existing flows still work and adding a new product requires only a new creator/product pair.
8. Explain tradeoffs.
   Call out added subclass complexity vs improved extensibility and decoupling.

## Decision Branches
- If there is only one stable product type and no extension pressure:
  Prefer direct construction; avoid pattern overhead.
- If creation logic needs multiple related factory methods:
  Consider evolving toward Abstract Factory.
- If object reuse/pooling is required:
  Allow factory method to return cached or pooled objects.
- If subtype explosion becomes excessive:
  Consider a temporary parameterized factory in base creator, then split only high-churn variants.

## Output Contract
When responding, provide:
1. A concise suitability verdict (why Factory Method or why not).
2. A mapped design with the four roles (Product, Concrete Product, Creator, Concrete Creator).
3. Step-by-step migration plan from current constructors to factory method.
4. A minimal code sketch in the user's language.
5. Validation checklist proving behavior parity and extension ease.

## Quality Checks
- Product interface covers behavior common to all variants.
- Creator business logic does not reference concrete product classes.
- New product type can be added without modifying existing client code.
- Factory method return type is abstraction (interface/base class), not concrete class.
- Tests cover at least one creator per variant and one client-level behavior test.

## Common Mistakes To Prevent
- Leaving hidden direct constructor calls in creator logic.
- Returning incompatible product types from creator subclasses.
- Turning creator into a pure factory with no business responsibility.
- Adding unnecessary subclass layers for low-variance code.

## References
- Detailed guide: [REFERENCE.md](./REFERENCE.md)
- Worked examples: [EXAMPLES.md](./EXAMPLES.md)
