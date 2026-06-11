---
name: clean-code
description: Apply Clean Code principles (meaningful names, small functions, error handling, TDD, SRP, smells & heuristics) when authoring, refactoring, or reviewing code—especially TypeScript. Use when the user asks for clean code, refactoring, extract function or class, code smell or maintainability review, readable tests, or Robert C. Martin's Clean Code rules.
---

# Clean Code

## Purpose
Guide writing and refactoring so code stays readable, testable, and easy to change—using Robert C. Martin's Clean Code habits: clear names, small units of behavior, honest error handling, focused tests, and continuous removal of duplication.

## Heuristics, not laws
Rules here are **defaults**, not absolutes. A short function with two related asserts, or a small `switch` on a stable discriminant, can be cleaner than forced polymorphism. Prefer **clarity and safe change** over checklist compliance; escalate splits and new types when **duplication or churn** justifies them.

## When To Use
Use this skill when the user asks to:
- **Review** existing code for smells, readability, or maintainability (e.g., "review this PR for clean code", "what smells do you see in `src/foo`?").
- **Refactor** toward simpler structure (e.g., "split this 200-line function", "make this module easier to test").
- **Author** new code with discipline (e.g., "implement X with TDD", "write a unit test that follows FIRST").
- Improve **naming**, **error handling**, **tests**, or **class boundaries** against Clean Code rules.

## When to ease off
Apply a **lighter** clean-code bar when the user signals urgency or disposability: production hotfix, time-boxed spike, or **generated/boilerplate** code they will not own. Still avoid silent failures and obvious lies in names; skip large extractions until there is breathing room.

## Operating Modes

### Authoring mode
Invoked while writing or refactoring. Flow: **pre-write checklist → implement smallest change → self-review → run tests**.

**Pre-write checklist (inline):**
1. Names and types express **domain intent** (no misleading containers or vague `data`/`handle`).
2. **Failing test first** when adding behavior (see [REFERENCE.md#unit-tests](REFERENCE.md#unit-tests)); otherwise, smallest change with a plan to add coverage.
3. **One level of abstraction** per function; queries/checks do not **mutate** hidden state.
4. **Error path** considered up front (typed or domain errors, `cause` where useful—not swallowed promises).
5. **Run the full suite** (or project equivalent) before calling the change done.

### Review mode
Invoked on existing code. Flow: **read top-down → tag findings against taxonomy → severity → concrete fix → cite [REFERENCE.md](REFERENCE.md)**.

## Authoring Workflow
1. **Name concepts first** — intention-revealing names for types, functions, and variables; replace magic numbers with named constants where they carry meaning.
2. **Write a failing unit test** (when adding behavior) — follow the Three Laws of TDD; keep the test about one concept (see [REFERENCE.md#unit-tests](REFERENCE.md#unit-tests)).
3. **Write the smallest production code** that passes the failing test—one level of abstraction per function; no sneaky side effects.
4. **Refactor** — remove duplication, clarify names, extract functions; prefer polymorphism over long `if`/`switch` chains when types vary (see [REFERENCE.md#functions](REFERENCE.md#functions)).
5. **Run all tests** — ensure green; repeat until the change is minimal and expressive (see [REFERENCE.md#emergence](REFERENCE.md#emergence)).

## Review Workflow
1. Read from **high level to detail** (public API → implementation).
2. Tag each issue with a **category** from the index below.
3. Assign **severity**: `blocker` (correctness/safety), `major` (maintainability risk), `minor` (clarity), `nit` (style only).
4. Propose a **concrete fix** (rename, extract, guard, test)—not vague advice.
5. **Cite** the relevant `REFERENCE.md` section anchor (e.g., `REFERENCE.md#error-handling`).

### Review severity rubric (examples)
Use judgment; these are typical mappings:

| Severity | Examples |
|----------|----------|
| `blocker` | Swallowed errors; missing auth/data checks; race or lost rejection on a critical path; API contract lies (types or names) |
| `major` | God class or 200-line multi-concern function; duplicated business rules; null/undefined soup without a documented boundary |
| `minor` | Unclear name; magic number with obvious domain meaning; comment instead of a name |
| `nit` | Formatting or subjective style that does not affect comprehension |

## Output Contract

### Authoring
- Implement with minimal commentary; add **brief rationale** only for non-obvious tradeoffs.
- Do not narrate line-by-line code the user can read.

### Review
Emit findings in this shape (one line per finding where possible):

`[severity] [category] path:line — finding → suggested fix (REFERENCE.md#anchor)`

Optional summary block at the end: counts by severity and top 3 recommended fixes.

## Quick Heuristics (highest leverage)
- **Functions**: small; **do one thing**; **one level of abstraction**; no hidden side effects in queries/checks.
- **Names**: reveal intent; avoid disinformation; **searchable** (no unexplained literals); length matches scope.
- **Errors (TypeScript)**: prefer **`throw` / `reject` with meaning** over numeric or boolean error codes; attach **`cause`** when wrapping; use **small custom `Error` subclasses** or tagged errors where it helps callers. On **`async` paths**, ensure rejections are handled—no empty `catch`, no unhandled floating promises.
- **Null and undefined**: pick a **project-consistent** “missing” story (`undefined` vs `null` vs result types); **do not widen** with `?? null` without reason; prefer **empty arrays/maps**, **`undefined`**, or explicit **`Result`-like** types over ambiguous absence.
- **Types**: use **discriminated unions** and **`never` exhaustiveness** (`switch` with default that narrows to `never`) so new variants break compile-time instead of silently falling through.
- **DRY**: factor duplication; delete commented-out dead code.
- **Conditionals**: **encapsulate** compound predicates and boundary checks; prefer polymorphism when **`switch`/`if` chains grow** with new variants—not when two branches are stable.
- **Tests**: **FIRST** (Fast, Independent, Repeatable, Self-validating, Timely); lean toward **one logical outcome / single concept** per test; production-quality naming in tests.
- **Classes**: **small**; **single responsibility**; **high cohesion** (what changes together stays together).
- **Systems**: separate **construction from use** (factories, DI, composition root); avoid arbitrary transitive navigation across objects.
- **Concurrency (TS)**: see [REFERENCE.md#concurrency](REFERENCE.md#concurrency) for async boundaries, shared mutable closures, and workers.

## Index of Detailed Topics
Deep rules and smell cues live in [REFERENCE.md](REFERENCE.md):

- [Meaningful names](REFERENCE.md#meaningful-names)
- [Functions](REFERENCE.md#functions)
- [Comments](REFERENCE.md#comments)
- [Error handling](REFERENCE.md#error-handling)
- [Unit tests](REFERENCE.md#unit-tests)
- [Classes](REFERENCE.md#classes)
- [Systems](REFERENCE.md#systems)
- [Emergence](REFERENCE.md#emergence)
- [Concurrency](REFERENCE.md#concurrency)
- [First make it work, then make it right](REFERENCE.md#first-make-it-work-then-make-it-right)
- [Smells and heuristics](REFERENCE.md#smells-and-heuristics)

Illustrative **TypeScript** refactors: [EXAMPLES.md](EXAMPLES.md).

## Validation Loop
- **After authoring**: run the full test suite (or project-equivalent); re-scan for duplication and misleading names.
- **After review fixes**: re-read changed regions; confirm severity dropped or finding resolved.

## Anti-Patterns to Flag Aggressively
- Names that **lie** (`List`, `Manager`, `Info` without meaning) or **encode** type or scope (`strName`, `m_`).
- **Comments** that excuse bad names or restate the obvious.
- **Functions** that both **check** and **mutate**, or do unrelated things in one block.
- **Returning null** or **passing null** without a tight, documented contract.
- **Hidden temporal coupling** (call order requirements not expressed in APIs).
- **Transitive navigation** (`getA().getB().getC()` across unrelated objects).
- **Magic numbers** without a named constant where meaning matters.
- **Multi-behavior unit tests** or tests that are not **FIRST**.
- **Large synchronized regions** (JVM), **unbounded async fan-out** without backpressure, or **shared mutable state** (closures, module singletons) without a clear ownership story (see [REFERENCE.md#concurrency](REFERENCE.md#concurrency)).
