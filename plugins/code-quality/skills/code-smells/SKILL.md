---
name: code-smells
description: Detect, name, and remediate code smells using the classic taxonomy (Bloaters, Object-Orientation Abusers, Change Preventers, Dispensables, Couplers) when reviewing, refactoring, or authoring code. Use when the user asks to find code smells, review code for maintainability, identify long methods/large classes/duplicate code/feature envy, or wants refactoring guidance tied to a named smell.
---

# Code Smells

## Purpose
Help spot, name, and treat **code smells** — surface indicators that usually correspond to deeper problems in a system. A smell is not a bug: the code works. It's a sign that the design will be costly to extend, understand, or change. Naming the smell precisely points to the right refactoring.

## What is a code smell?
> "Code smells are certain structures in the code that indicate violation of fundamental design principles and negatively impact design quality."

Smells don't crop up all at once — they **accumulate over time** as a program evolves, especially when nobody actively removes them. Treat them as heuristics that warrant a closer look, not automatic defects: context decides whether a given smell is worth fixing now.

## When To Use
Use this skill when the user asks to:
- **Find smells** in a file, module, or PR ("what smells do you see here?", "review this for code smells").
- **Refactor** code and wants the change anchored to a named smell and its standard treatment.
- **Author** code and wants a pre-merge check against common smells.
- Diagnose **why a change is painful** (e.g., "every time I touch X I have to edit five files").

## When to ease off
Smells are **defaults, not laws**. Apply a lighter bar when the user signals urgency or disposability: production hotfix, time-boxed spike, or generated/boilerplate code they won't own. A short method, a small `switch` on a stable discriminant, or a tiny data holder can be perfectly fine. Prefer **clarity and safe change** over checklist compliance; escalate a treatment only when **duplication, churn, or comprehension cost** justifies it.

## The Five Categories
Each smell belongs to one family. Detail for every smell — signs & symptoms, reasons it occurs, treatment (refactorings), and payoff — lives in [REFERENCE.md](REFERENCE.md).

### Bloaters
Code, methods, and classes that have grown to unmanageable proportions. They accumulate gradually.
- [Long Method](REFERENCE.md#long-method)
- [Large Class](REFERENCE.md#large-class)
- [Primitive Obsession](REFERENCE.md#primitive-obsession)
- [Long Parameter List](REFERENCE.md#long-parameter-list)
- [Data Clumps](REFERENCE.md#data-clumps)

### Object-Orientation Abusers
Incomplete or incorrect application of object-oriented principles.
- [Alternative Classes with Different Interfaces](REFERENCE.md#alternative-classes-with-different-interfaces)
- [Refused Bequest](REFERENCE.md#refused-bequest)
- [Switch Statements](REFERENCE.md#switch-statements)
- [Temporary Field](REFERENCE.md#temporary-field)

### Change Preventers
A change in one place forces many changes elsewhere — development becomes slow and expensive.
- [Divergent Change](REFERENCE.md#divergent-change)
- [Parallel Inheritance Hierarchies](REFERENCE.md#parallel-inheritance-hierarchies)
- [Shotgun Surgery](REFERENCE.md#shotgun-surgery)

### Dispensables
Something pointless and unneeded whose removal makes code cleaner and easier to understand.
- [Comments](REFERENCE.md#comments)
- [Duplicate Code](REFERENCE.md#duplicate-code)
- [Data Class](REFERENCE.md#data-class)
- [Dead Code](REFERENCE.md#dead-code)
- [Lazy Class](REFERENCE.md#lazy-class)
- [Speculative Generality](REFERENCE.md#speculative-generality)

### Couplers
Excessive coupling between classes, or what happens when coupling is replaced by excessive delegation.
- [Feature Envy](REFERENCE.md#feature-envy)
- [Inappropriate Intimacy](REFERENCE.md#inappropriate-intimacy)
- [Incomplete Library Class](REFERENCE.md#incomplete-library-class)
- [Message Chains](REFERENCE.md#message-chains)
- [Middle Man](REFERENCE.md#middle-man)

## Review Workflow
1. Read from **high level to detail** (public API → implementation).
2. Tag each issue with a **smell name** and its **category** from the index above.
3. Assign **severity**: `blocker` (correctness/safety risk), `major` (maintainability risk), `minor` (clarity), `nit` (style only). Most smells are `major`/`minor`, not `blocker`.
4. Propose the **standard treatment** (the named refactoring), not vague advice.
5. **Cite** the relevant `REFERENCE.md` section anchor (e.g., `REFERENCE.md#feature-envy`).

## Output Contract
Emit findings one line each where possible:

`[severity] [smell] path:line — what's wrong → suggested refactoring (REFERENCE.md#anchor)`

Optional summary at the end: counts by category and the top 3 recommended refactorings.

## Quick Heuristics (highest leverage)
- **A method longer than ~10 lines** or that needs a comment to explain a block → suspect Long Method.
- **A class with many fields/methods doing several jobs** → Large Class / Divergent Change.
- **The same group of fields/params appearing together repeatedly** → Data Clumps / Long Parameter List.
- **Primitives standing in for concepts** (strings for currency, ints for state) → Primitive Obsession.
- **A method more interested in another class's data than its own** → Feature Envy.
- **`a.getB().getC().getD()`** → Message Chains.
- **One change forces edits in many classes** → Shotgun Surgery; **one class changed for many reasons** → Divergent Change.
- **Adding a subclass forces a parallel subclass elsewhere** → Parallel Inheritance Hierarchies.
- **`switch`/`if` chains on a type code that recur** → Switch Statements.

## Anti-Patterns to Flag Aggressively
- Duplicated business rules across files (Duplicate Code).
- Commented-out code and dead branches (Dead Code).
- Comments that excuse bad names instead of fixing them (Comments).
- Subclasses that throw/no-op inherited methods they don't want (Refused Bequest).
- Classes built for imagined future needs that never arrived (Speculative Generality).
- Fields only populated in certain circumstances (Temporary Field).
