# Pragmatic Programmer — Reference

Compact rules and **smell cues** for DRY and Orthogonality. Pair with [SKILL.md](SKILL.md) and [EXAMPLES.md](EXAMPLES.md).

## Table of contents
- [DRY — types of duplication](#dry--types-of-duplication)
- [DRY — beyond code](#dry--beyond-code)
- [DRY — making reuse easy](#dry--making-reuse-easy)
- [Orthogonality — decoupling](#orthogonality--decoupling)
- [Orthogonality — global state](#orthogonality--global-state)
- [Orthogonality — similar functions](#orthogonality--similar-functions)
- [Combined: where DRY and Orthogonality meet](#combined-where-dry-and-orthogonality-meet)
- [Smells and heuristics table](#smells-and-heuristics-table)

---

## DRY — types of duplication

Every piece of **knowledge** must have a single, unambiguous, authoritative representation within a system.

The Pragmatic Programmer identifies four kinds of duplication:

### 1. Imposed duplication
The environment or tooling seems to require it (two languages, a generated file, a protocol spec). Push back:
- Use code generation to derive one form from a single source.
- Use a schema (e.g., JSON Schema, Zod, Prisma) and derive both type and validator from it.
- **Smell to flag:** hand-maintaining a TypeScript type and a separate runtime validator that share the same shape.

### 2. Inadvertent duplication
Developers don't realize they are duplicating — often a design problem where two concepts that seem different are actually the same rule expressed twice.
- **Smell to flag:** `OrderValidator` and `CartValidator` both implement the same "no negative quantity" check with slightly different wording.

### 3. Impatient duplication
Shortcuts taken under time pressure — "I'll just copy this and tweak it."
- This is the most common and most dangerous: the copy will diverge silently.
- **Smell to flag:** two functions that are identical except for one constant or one field name.

### 4. Interdeveloper duplication
Multiple team members independently build the same utility because nobody made it discoverable.
- The fix is communication and making reuse easy, not enforcement alone.
- **Smell to flag:** three files each implementing their own `formatCurrency` or `parseDate`.

---

## DRY — beyond code

DRY applies to **knowledge**, not just lines of code. Any place a fact about the system is expressed is a potential violation:

| Location | Duplication example | Fix |
|----------|---------------------|-----|
| **Documentation** | Comment restates what the function name already says | Remove the comment; improve the name |
| **Tests** | Test data hard-codes a limit that's already a named constant in production code | Reference the constant in tests |
| **Config** | Timeout value in `config.yaml` and also as a fallback default in code | One source; code reads config, provides no magic fallback |
| **Schema + type** | Zod schema and a TypeScript `interface` with the same fields | Derive the type from the schema: `z.infer<typeof schema>` |
| **Database + domain** | DB column name and domain field name maintained separately | Use an ORM/mapper with a single field definition |

**Smell to flag:** documentation that will be wrong six months after the code changes, because they are two separate representations of the same knowledge.

---

## DRY — making reuse easy

If reuse is hard, people duplicate. Fix the friction:

- **Discoverable utilities**: a `utils/` or `shared/` module with clear names so teammates can find what exists.
- **Low coupling in helpers**: utility functions that don't depend on the full application context are easier to reuse.
- **Clear ownership**: a rule with a clear owner gets updated in one place when requirements change.

**Smell to flag:** a shared abstraction that requires importing half the application context — people will inline instead.

---

## Orthogonality — decoupling

Two components are **orthogonal** if changing one does not require changing the other. Aim for "shy code": modules that reveal only what callers need and depend only on what they actually use.

### Keep components independent
- Design so that one module's changes stay local. If a bug fix requires editing three files, the design has a coupling problem.
- Use interfaces or event boundaries so concrete implementations don't leak.
- **Smell to flag:** changing the HTTP response shape requires updating both the controller *and* the domain entity.

### Law of Demeter (tell, don't ask)
Call only methods of: the object itself, parameters passed in, objects it created, or direct component objects. Avoid deep chains.
- **Smell to flag:** `order.getCustomer().getAddress().getCity()` — couples caller to three internal structures.

### Layering
Infrastructure details (SQL, HTTP status codes, file paths) must not appear in domain logic. Each layer speaks only to its immediate neighbor.
- **Smell to flag:** a `User` domain class that knows about `res.status(404)` or `SELECT` queries.

---

## Orthogonality — global state

Global mutable state is the opposite of orthogonal: every module that reads or writes it is secretly coupled to every other module that touches it.

### Rules
- **Prefer scoped, owned state**: pass state explicitly through constructors or function parameters.
- **Module-level singletons**: only acceptable when they are truly stateless (pure configuration read at startup, never mutated).
- **Mutable module-level caches**: give them a clear owner; expose them only through a controlled interface.
- **Smell to flag:** `let currentUser: User | null = null` at module scope, written by auth middleware and read by unrelated business logic.

### Why it matters for orthogonality
Two modules that both read and write the same global are coupled through that shared state. A change to how one module writes the value can break the other without any visible dependency in the code.

---

## Orthogonality — similar functions

Near-duplicate functions are a symptom: two functions that differ in one constant, one field, or one tiny behavior indicate a missing parameterized abstraction.

### Rules
- **Extract the common core**: identify what differs and make it a parameter (value, callback, strategy).
- **Don't unify unrelated things**: if the functions have different **reasons to change**, keep them separate. Forced unification creates its own coupling.
- **Smell to flag**: two functions whose bodies are 90% identical, differing only in a status string or a field name.

### Distinguish accidental from intentional similarity
- **Accidental**: `sendWelcomeEmail` and `sendPasswordResetEmail` share the same SMTP setup boilerplate — extract a `sendEmail(template, recipient)` base.
- **Intentional**: `calculateTax` and `calculateDiscount` happen to have similar math but represent different business rules — keep them separate.

---

## Combined: where DRY and Orthogonality meet

The two principles reinforce each other:

| Violation | DRY angle | Orthogonality angle |
|-----------|-----------|---------------------|
| Business rule in domain + test + docs | Three representations of one fact | All three must change together → coupling |
| Global config read everywhere | Multiple readers of one "truth" | Any module change to config format breaks all readers |
| Copy-pasted validation | Same rule expressed twice | Both copies must be updated for one business change |
| Parallel if/switch on same type | Same type-dispatch logic duplicated | Two call sites coupled to the same type enum |

**The combined heuristic**: *One change, one place.* If a single business decision requires touching more than one file, either DRY or Orthogonality (or both) has been violated.

---

## Smells and heuristics table

| Smell | Principle | Rule | Fix |
|-------|-----------|------|-----|
| Copy-paste with tiny edits | DRY — impatient | Knowledge must have one representation | Extract parameterized function or shared constant |
| Same validation in two layers | DRY — inadvertent | Rule belongs in one authoritative place | Move to domain layer; call from both sides |
| Type and runtime validator in sync | DRY — imposed | Derive one from the other | `z.infer<>`, Prisma type generation, etc. |
| Comment restates the code | DRY — documentation | Code *is* the representation; comment is a duplicate | Remove comment; improve name |
| Magic number repeated in 3 files | DRY | Constants must have one home | Named constant in a shared module |
| `a.getB().getC().getD()` chains | Orthogonality — decoupling | Don't navigate transitive internals | Ask the closest object for what you need |
| HTTP status in domain class | Orthogonality — layering | Infrastructure must not leak into domain | Map at the boundary (controller/adapter) |
| Global mutable `let` at module scope | Orthogonality — global state | State should have a scoped owner | Pass via constructor or function parameter |
| Two near-identical functions | Orthogonality — similar functions | Extract the differing part as a parameter | Parameterized function or strategy |
| Parallel switch trees on same enum | DRY + Orthogonality | One type change forces two edits | Polymorphism or a single dispatch table |
| Test hard-codes production constant | DRY — tests | Tests are knowledge too | Import and reference the constant |
| Three teams each wrote `parseDate` | DRY — interdeveloper | Make reuse easy and discoverable | Shared utility, communicated to the team |

---

End of reference. Return to [SKILL.md](SKILL.md) for workflows and [EXAMPLES.md](EXAMPLES.md) for before/after patterns.
