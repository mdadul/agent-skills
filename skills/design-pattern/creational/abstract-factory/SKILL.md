---
name: abstract-factory
description: Teach and apply the Abstract Factory pattern to create families of related products through abstract interfaces while keeping variants compatible. Use when users ask about Abstract Factory, replacing concrete constructors for multiple product types, ensuring variant consistency, or comparing Abstract Factory with Factory Method or Builder.
---

# Abstract Factory

## Purpose
Help the user decide whether Abstract Factory is the right fit, then design and apply it with low-risk, incremental refactoring.

## When To Use
- User asks to explain Abstract Factory or to design family-based object creation.
- Code creates multiple related product types and must keep them variant-compatible.
- Client code should avoid dependencies on concrete factory and product classes.
- New product variants (themes/platforms/vendors) are expected over time.
- Existing class has many factory methods and creation concerns are blurring responsibilities.

## Inputs
- Product family matrix (distinct product types x variants).
- Existing constructor hotspots in client or setup code.
- Compatibility constraints between products within the same variant.
- Runtime selection rules (configuration, environment, tenant, feature flags).
- Language/framework constraints and test expectations.

## Workflow
1. Clarify the goal.
   Confirm whether the user wants concept explanation, architecture review, or implementation/refactor.
2. Check applicability.
   Use Abstract Factory when multiple related product types must be created as a coherent variant.
3. Map the family matrix.
   List product types (for example `Button`, `Checkbox`) and variants (for example `Win`, `Mac`).
4. Define abstractions.
   Create abstract product interfaces for each product type and an abstract factory interface with creation methods for all product types.
5. Implement concrete variants.
   Create one concrete factory per variant; each factory returns only products from that same variant.
6. Centralize factory selection.
   Choose concrete factory during initialization based on configuration/environment, then inject it.
7. Replace direct constructors.
   Swap client-side `new` calls for abstract factory method calls.
8. Validate compatibility and extensibility.
   Verify products from the same factory collaborate correctly and new variants require additive changes only.

## Decision Branches
- If there is only one product type and variation is limited:
  Prefer Factory Method instead of Abstract Factory.
- If object construction is step-by-step with optional assembly stages:
  Prefer Builder.
- If adding a new product type is frequent and costly across all variants:
  Re-evaluate matrix stability; Abstract Factory may increase maintenance overhead.
- If runtime compatibility between product types is critical:
  Strongly favor Abstract Factory to enforce variant coherence.

## Output Contract
When responding, provide:
1. A suitability verdict (use Abstract Factory or choose a simpler alternative).
2. A family matrix summary (product types and variants).
3. Role mapping (Abstract Products, Concrete Products, Abstract Factory, Concrete Factories, Client wiring).
4. Incremental migration plan from constructor calls to factory usage.
5. Minimal code sketch in the user's language, including initialization-time factory selection.
6. Validation checklist proving compatibility and extensibility behavior.

## Quality Checks
- Every product type has an abstract interface.
- Abstract factory includes creation methods for all product types in scope.
- Concrete factory creates only one coherent variant family.
- Client depends only on abstract product and factory interfaces.
- Factory selection is centralized at app initialization/composition root.
- Adding a new variant does not require changing client business logic.

## Common Mistakes To Prevent
- Mixing product variants inside one concrete factory.
- Leaking concrete product types into client code.
- Skipping product interfaces and returning concrete types from abstract factory API.
- Using Abstract Factory when only one product type exists.
- Spreading factory selection logic across multiple runtime call sites.

## References
- Detailed guide: [REFERENCE.md](./REFERENCE.md)
- Worked examples: [EXAMPLES.md](./EXAMPLES.md)
