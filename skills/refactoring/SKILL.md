---
name: refactoring
description: Apply named refactoring techniques (Extract Method, Move Method, Replace Conditional with Polymorphism, etc.) organized by the classic catalog — Composing Methods, Moving Features between Objects, Organizing Data, Simplifying Conditional Expressions, Simplifying Method Calls, and Dealing with Generalization. Use when the user wants to refactor code, restructure without changing behavior, clean up a method/class/hierarchy, or asks how to perform a specific named refactoring or treat a code smell.
---

# Refactoring Techniques

## Purpose
Refactoring is a **controlled technique for improving the design of existing code** — a series of small, behavior-preserving transformations. Each individual change is tiny, but a sequence of them can significantly restructure code while keeping it working at every step. This skill catalogs the named techniques and tells you when and how to apply each.

## What is refactoring?
> Refactoring changes the *internal structure* of code without changing its *external behavior*.

Two rules of the discipline:
1. **Don't refactor and add features at the same time.** Separate the two into distinct steps (and ideally distinct commits).
2. **Keep tests green.** Have working tests before you start; run them after each small step. If something breaks, you know the last tiny change caused it.

Refactor when code smells warrant it (see the companion **code-smells** skill), before adding a feature to ground you're about to touch, or when reviewing makes code hard to understand.

## When To Use
Use this skill when the user asks to:
- **Refactor** code or restructure it without changing behavior.
- Perform a **named technique** ("extract this into a method", "pull this field up", "replace this switch with polymorphism").
- **Treat a code smell** and wants the concrete mechanics of the fix (pairs with the code-smells skill).
- Improve a **method, class, or inheritance hierarchy** for readability, testability, or reuse.

## When to ease off
Refactoring is an investment, not an end in itself. Apply a lighter bar when the code is disposable (spike, generated boilerplate), when there's no test safety net and adding one isn't feasible right now, or under a time-critical hotfix. Don't refactor code you're about to delete. Prefer the smallest change that removes the pain over a sweeping restructure.

## The Six Groups
Mechanics for every technique live in [REFERENCE.md](REFERENCE.md).

### Composing Methods
Streamline methods, remove duplication, pave the way for further change. Overly long methods are the root of most evil.
- [Extract Method](REFERENCE.md#extract-method) · [Inline Method](REFERENCE.md#inline-method) · [Extract Variable](REFERENCE.md#extract-variable) · [Inline Temp](REFERENCE.md#inline-temp) · [Replace Temp with Query](REFERENCE.md#replace-temp-with-query) · [Split Temporary Variable](REFERENCE.md#split-temporary-variable) · [Remove Assignments to Parameters](REFERENCE.md#remove-assignments-to-parameters) · [Replace Method with Method Object](REFERENCE.md#replace-method-with-method-object) · [Substitute Algorithm](REFERENCE.md#substitute-algorithm)

### Moving Features between Objects
Safely move functionality between classes, create new classes, hide implementation details.
- [Move Method](REFERENCE.md#move-method) · [Move Field](REFERENCE.md#move-field) · [Extract Class](REFERENCE.md#extract-class) · [Inline Class](REFERENCE.md#inline-class) · [Hide Delegate](REFERENCE.md#hide-delegate) · [Remove Middle Man](REFERENCE.md#remove-middle-man) · [Introduce Foreign Method](REFERENCE.md#introduce-foreign-method) · [Introduce Local Extension](REFERENCE.md#introduce-local-extension)

### Organizing Data
Handle data better — replace primitives with rich classes, untangle class associations for portability and reuse.
- [Change Value to Reference](REFERENCE.md#change-value-to-reference) · [Change Reference to Value](REFERENCE.md#change-reference-to-value) · [Duplicate Observed Data](REFERENCE.md#duplicate-observed-data) · [Self Encapsulate Field](REFERENCE.md#self-encapsulate-field) · [Replace Data Value with Object](REFERENCE.md#replace-data-value-with-object) · [Replace Array with Object](REFERENCE.md#replace-array-with-object) · [Change Unidirectional Association to Bidirectional](REFERENCE.md#change-unidirectional-association-to-bidirectional) · [Change Bidirectional Association to Unidirectional](REFERENCE.md#change-bidirectional-association-to-unidirectional) · [Encapsulate Field](REFERENCE.md#encapsulate-field) · [Encapsulate Collection](REFERENCE.md#encapsulate-collection) · [Replace Magic Number with Symbolic Constant](REFERENCE.md#replace-magic-number-with-symbolic-constant) · [Replace Type Code with Class](REFERENCE.md#replace-type-code-with-class) · [Replace Type Code with Subclasses](REFERENCE.md#replace-type-code-with-subclasses) · [Replace Type Code with State/Strategy](REFERENCE.md#replace-type-code-with-statestrategy) · [Replace Subclass with Fields](REFERENCE.md#replace-subclass-with-fields)

### Simplifying Conditional Expressions
Conditionals accumulate complexity over time; these techniques fight back.
- [Consolidate Conditional Expression](REFERENCE.md#consolidate-conditional-expression) · [Consolidate Duplicate Conditional Fragments](REFERENCE.md#consolidate-duplicate-conditional-fragments) · [Decompose Conditional](REFERENCE.md#decompose-conditional) · [Replace Conditional with Polymorphism](REFERENCE.md#replace-conditional-with-polymorphism) · [Remove Control Flag](REFERENCE.md#remove-control-flag) · [Replace Nested Conditional with Guard Clauses](REFERENCE.md#replace-nested-conditional-with-guard-clauses) · [Introduce Null Object](REFERENCE.md#introduce-null-object) · [Introduce Assertion](REFERENCE.md#introduce-assertion)

### Simplifying Method Calls
Make calls simpler and interfaces between classes easier to understand.
- [Add Parameter](REFERENCE.md#add-parameter) · [Remove Parameter](REFERENCE.md#remove-parameter) · [Rename Method](REFERENCE.md#rename-method) · [Separate Query from Modifier](REFERENCE.md#separate-query-from-modifier) · [Parameterize Method](REFERENCE.md#parameterize-method) · [Introduce Parameter Object](REFERENCE.md#introduce-parameter-object) · [Preserve Whole Object](REFERENCE.md#preserve-whole-object) · [Remove Setting Method](REFERENCE.md#remove-setting-method) · [Replace Parameter with Explicit Methods](REFERENCE.md#replace-parameter-with-explicit-methods) · [Replace Parameter with Method Call](REFERENCE.md#replace-parameter-with-method-call) · [Hide Method](REFERENCE.md#hide-method) · [Replace Constructor with Factory Method](REFERENCE.md#replace-constructor-with-factory-method) · [Replace Error Code with Exception](REFERENCE.md#replace-error-code-with-exception) · [Replace Exception with Test](REFERENCE.md#replace-exception-with-test)

### Dealing with Generalization
Move functionality along the inheritance hierarchy; create classes and interfaces; trade inheritance for delegation and back.
- [Pull Up Field](REFERENCE.md#pull-up-field) · [Pull Up Method](REFERENCE.md#pull-up-method) · [Pull Up Constructor Body](REFERENCE.md#pull-up-constructor-body) · [Push Down Field](REFERENCE.md#push-down-field) · [Push Down Method](REFERENCE.md#push-down-method) · [Extract Subclass](REFERENCE.md#extract-subclass) · [Extract Superclass](REFERENCE.md#extract-superclass) · [Extract Interface](REFERENCE.md#extract-interface) · [Collapse Hierarchy](REFERENCE.md#collapse-hierarchy) · [Form Template Method](REFERENCE.md#form-template-method) · [Replace Inheritance with Delegation](REFERENCE.md#replace-inheritance-with-delegation) · [Replace Delegation with Inheritance](REFERENCE.md#replace-delegation-with-inheritance)

## Workflow
1. **Ensure a test safety net.** Confirm tests exist and pass before touching anything. If not, add characterization tests first.
2. **Identify the target** — name the smell or the structural problem (cite the code-smells skill).
3. **Pick the technique** from the catalog that addresses it.
4. **Apply in small steps**, following the mechanics in [REFERENCE.md](REFERENCE.md). Compile/run after each step.
5. **Run tests** after every step; if red, revert the last step.
6. **Commit** the refactoring separately from any feature work.

## Output Contract
When proposing a refactoring, state:

`[technique] target → why (smell it removes) → ordered mechanics (REFERENCE.md#anchor)`

Keep behavior identical. Call out explicitly if a step is *not* behavior-preserving (rare) so the user can add coverage.

## Relationship to Code Smells
Smells tell you **what** is wrong; refactorings tell you **how** to fix it. The companion **code-smells** skill names each smell's treatments using the techniques cataloged here. When asked to "fix this smell", diagnose with code-smells, then apply the matching technique from this skill.
