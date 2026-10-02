---
name: pragmatic-programmer
description: Apply Pragmatic Programmer principles—DRY (Don't Repeat Yourself) and Orthogonality—when authoring, reviewing, or refactoring code. Detects knowledge duplication, tight coupling, global state abuse, and similar-function sprawl, then proposes authoritative single-source fixes. Use when the user asks for DRY review, coupling analysis, orthogonality check, "make this more pragmatic", or references The Pragmatic Programmer book principles.
---

# Pragmatic Programmer

## Purpose
Apply the two foundational principles from *The Pragmatic Programmer* — DRY and Orthogonality — to keep systems easy to understand, change, and extend without fear.

## Heuristics, not laws
These are **defaults**, not absolutes. A short inline duplication that keeps context local is cleaner than a forced abstraction. Coupling is sometimes intentional (a transaction boundary must bind things). Prefer **clarity and safe change** over checklist compliance.

## When To Use
- **Review** code for DRY violations, coupling, global state, or similar-function sprawl.
- **Refactor** toward a single authoritative source of truth.
- **Author** new code with orthogonal, loosely-coupled design.
- User mentions: "DRY", "duplication", "coupling", "orthogonal", "global state", "similar functions", "single source of truth", "Pragmatic Programmer".

## Operating Modes

### Authoring mode
Flow: **pre-write checklist → implement → self-review → validate**.

Pre-write checklist:
1. Can this knowledge (rule, format, schema, constant) be expressed **once**? Locate the authoritative source first.
2. Do the components I'm connecting **need** to know about each other, or can I introduce a thin interface or event?
3. Am I touching global state? If yes, is there a scoped owner instead?
4. Is this function **meaningfully different** from an existing one, or a near-copy that belongs in a shared abstraction?

### Review mode
Flow: **read top-down → tag findings → severity → concrete fix → cite [REFERENCE.md](REFERENCE.md)**.

## Authoring Workflow
1. **Locate knowledge** — identify every concept, rule, schema, and constant. Each must live in exactly one place.
2. **Draw component boundaries** — components should hide their internals. If changing A requires changing B, they are too coupled.
3. **Push state inward** — replace global/module-level mutable state with scoped, owned state passed explicitly.
4. **Unify similar functions** — when two functions differ only in a value or a small behavior, extract a parameterized common form.
5. **Validate** — confirm no duplicate logic remains and that changes to one component do not ripple unexpectedly.

## Review Workflow
1. Read from **high level to detail** (module exports → implementation).
2. Tag each finding with a **principle**: `DRY` or `Orthogonality`.
3. Assign **severity**: `blocker` (correctness/divergence risk), `major` (maintenance tax), `minor` (clarity), `nit`.
4. Propose a **concrete fix** — not vague advice.
5. Cite the relevant `REFERENCE.md` anchor.

### Review severity rubric

| Severity | Examples |
|----------|----------|
| `blocker` | Business rule expressed in two places that can diverge; shared global state mutated by unrelated modules |
| `major` | Copy-pasted validation logic; near-identical functions with no shared core; God module imports half the system |
| `minor` | Magic constant repeated in 2 places; two tiny helpers that could be one parameterized function |
| `nit` | Cosmetic repetition that carries no divergence risk |

## Output Contract

### Authoring
- Implement with minimal commentary. Add a brief rationale only for non-obvious orthogonality tradeoffs.

### Review
Emit findings in this shape:

`[severity] [DRY|Orthogonality] path:line — finding → suggested fix (REFERENCE.md#anchor)`

Optional summary block: counts by severity and top 3 recommended fixes.

## Quick Heuristics

### DRY
- **Knowledge, not just code** — duplication of business rules in tests, schemas, docs, and configs counts.
- **Single authoritative source** — every constant, formula, and validation rule lives in exactly one place.
- **Make reuse easy** — if teammates duplicate because reuse is hard, fix the friction, not just the duplicate.
- **Four duplication types to watch**: imposed (forced by tooling), inadvertent (unintentional), impatient (shortcut), interdeveloper (team ignorance).

### Orthogonality
- **Shy code** — modules should not reveal unnecessary internals and should not depend on other modules' implementations.
- **No global data** — global/module-level mutable state silently couples every reader and writer.
- **No similar functions** — near-duplicate functions signal a missing abstraction, not a naming problem.
- **One change, one place** — if fixing a bug requires edits in three files, the design has a coupling problem.
- **Layer cleanly** — infrastructure details (HTTP, DB) must not leak into domain logic.

## Index of Detailed Topics
Deep rules and smell cues live in [REFERENCE.md](REFERENCE.md):

- [DRY — types of duplication](REFERENCE.md#dry--types-of-duplication)
- [DRY — beyond code](REFERENCE.md#dry--beyond-code)
- [DRY — making reuse easy](REFERENCE.md#dry--making-reuse-easy)
- [Orthogonality — decoupling](REFERENCE.md#orthogonality--decoupling)
- [Orthogonality — global state](REFERENCE.md#orthogonality--global-state)
- [Orthogonality — similar functions](REFERENCE.md#orthogonality--similar-functions)
- [Combined: where DRY and Orthogonality meet](REFERENCE.md#combined-where-dry-and-orthogonality-meet)
- [Smells and heuristics table](REFERENCE.md#smells-and-heuristics-table)

Illustrative **TypeScript** refactors: [EXAMPLES.md](EXAMPLES.md).

## Validation Loop
- **After authoring**: confirm each concept appears once; draw a dependency graph mentally — no surprise edges.
- **After review fixes**: re-read changed regions; confirm severity dropped or finding resolved.

## Anti-Patterns to Flag Aggressively
- Business rule expressed in **two or more places** — tests, docs, code, or config all count.
- **Copy-paste with tiny edits** — the canonical signal of impatient duplication.
- **Global mutable singletons** accessed from unrelated modules.
- **Parallel switch/if trees** on the same discriminant — change one, must change the other.
- **Near-identical functions** differing only in a constant or a single behavior branch.
- **Leaking infrastructure** (SQL, HTTP status, file paths) into domain objects.
- **Documentation that restates code** — it will diverge; make the code speak instead.
- **Schema defined twice** — e.g., TypeScript type and a separate runtime validator with the same shape.
