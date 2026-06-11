# Clean Code — Reference

Compact rules and **smell cues** for review and authoring. Pair with [SKILL.md](SKILL.md) and [EXAMPLES.md](EXAMPLES.md).

## Table of contents
- [Meaningful names](#meaningful-names)
- [Functions](#functions)
- [Comments](#comments)
- [Error handling](#error-handling)
- [Unit tests](#unit-tests)
- [Classes](#classes)
- [Systems](#systems)
- [Emergence](#emergence)
- [Concurrency](#concurrency)
- [First make it work, then make it right](#first-make-it-work-then-make-it-right)
- [Smells and heuristics](#smells-and-heuristics)

## Meaningful names

### Use intention-revealing names
Names should answer *why* something exists, *what* it does, and *how* it is used. Prefer domain vocabulary the team already uses.
- **Smell to flag:** `d`, `data`, `ret`, `tmp`, `theInfo` with no domain meaning.

### Avoid disinformation
Do not encode types or container kinds in the name if they are wrong (e.g., `accountList` for a set or map). Avoid names that look like other concepts (`O` vs `0`, `l` vs `1`).
- **Smell to flag:** name says "List" but is not a list; plural for a single value; abbreviations that read as different words.

### Make meaningful distinction
`a1`/`a2` or `data1`/`data2` is not a distinction—use *semantic* difference (`source` vs `destination`, `gross` vs `net`).
- **Smell to flag:** numbered suffixes, `x` vs `x2` for "the other one."

### Use pronounceable names
If you cannot say it in a code review, you will not discuss it clearly.
- **Smell to flag:** consonant soup (`genymdhms`), unpronounceable acronyms without team agreement.

### Use searchable names
Single-letter names and magic literals are hard to find. Use named constants for numbers and strings with business meaning; use full words for important concepts.
- **Smell to flag:** `7` (days in a week in business rules), `42` (status), repeated string literals for the same concept.

### Avoid encodings
Drop unnecessary Hungarian notation, member prefixes, and type encodings; let the type system and small scopes carry that information.
- **Smell to flag:** `strName`, `m_count`, `IInterface` on every name.

### Names and scope
Prefer **longer names for longer scopes**; very short names are tolerable in tiny blocks (loop indices) if still clear.
- **Smell to flag:** `i` used far from the loop; 3-character names in module-level APIs.

### Standard nomenclature
Use the names your language and domain community already use (`Factory`, `Visitor`, `Controller` when they match the pattern).
- **Smell to flag:** inventing a new word for a well-known pattern or library concept.

## Functions

### Small
Aim for functions that fit in a small window; if you cannot name it in a short phrase, it may be doing too much. Use the team’s line-count norm as a soft cap, not a religion.
- **Smell to flag:** 100+ line function, many blank sections, "and also" in the function name.

### Do one thing
A function should do *one* thing, do it well, and do it only at *one* level of abstraction. Mixing high-level policy with low-level detail in one function is a split signal.
- **Smell to flag:** `process` + `openFile` + `writeHttp` in one function; `validateAndSaveAndNotify`.

### Have no side effects
A "get" or "check" should not mutate state. A function that looks like a query but changes data is a trap.
- **Smell to flag:** `isValid` that also sets a session; `getX` that lazily mutates a cache without a name hint.

### Don’t repeat yourself (DRY)
Each piece of knowledge should have a single, authoritative expression. Copy-paste with tiny edits is a maintenance tax.
- **Smell to flag:** three near-identical `try/catch` blocks; three `if` trees with the same structure.

### One level of abstraction
Statements in a function should all read at the same level (all "what", or all "how" for a small inner step—not both interleaved).
- **Smell to flag:** a function that both iterates a list and byte-mangles strings in the same block.

### Prefer polymorphism to if/else or switch/case
When behavior varies by *type* and the `switch` grows, use polymorphism (strategies, visitors, method dispatch) so new types do not break a central switch.
- **Smell to flag:** `switch (type)` with new `case` every release; long `if/else` chains on the same discriminator.

### Encapsulate conditionals and boundaries
Replace compound predicates with well-named functions (`shouldBeDeleted(employee)`). Capture boundary rules (`offByOne`) once.
- **Smell to flag:** repeated `if (x >= 0 && x < length)` everywhere; magic `+1`/`-1` scattered.

### Functions should say what they do
Name after behavior, not commentary. If the name needs "And" or "Or," consider splitting.
- **Smell to flag:** `doIt`, `handle`, `manage`, `process` without a verb-object story.

## Comments

### Good comments
- **Informative:** explains a non-obvious constraint (legal, performance, protocol).
- **Intent:** why this approach exists when alternatives were rejected.
- **Clarification:** translates obscure API or domain jargon—prefer fixing the API/name first.
- **Warning of consequences:** documents risk (e.g., order-dependent, must run on UI thread).
- **Amplification:** emphasizes importance when easy to miss.

### Bad comments
Comments are not an excuse for bad names. Avoid redundant narration, commented-out code, and journals of change (use version control).
- **Smell to flag:** comment duplicates the next line; comment says "hack" with no ticket or date; blocks of dead code.

### Don’t put too much information
Long essays belong in docs; keep comments local and scannable.
- **Smell to flag:** tutorial pasted above a 5-line function.

## Error handling

### Use exceptions rather than return codes
Prefer throwing/handing errors through the language’s exception mechanism so callers cannot ignore failures silently.
- **Smell to flag:** `int` return codes checked inconsistently; boolean `false` meaning "error, reason unknown."

### Write try-catch-finally first
Sketch the happy path and error path together so resources are released and failures are visible early.
- **Smell to flag:** `catch` blocks added as afterthoughts; swallowed exceptions.

### Provide context with exceptions
Wrap low-level errors with domain meaning (what failed, for whom, with what inputs)—without leaking secrets.
- **Smell to flag:** generic `Error` with no cause chain; logging only `"failed"` without operation name.

### Don’t return null
Return empty collections, optional types, or explicit result objects so callers do not forget null checks. In **TypeScript**, prefer a **single** “missing” convention (`undefined`, `null`, or a `Result` type—team-wide); avoid `?? null` that only **widens** a type without clarifying intent.
- **Smell to flag:** "returns null if not found" everywhere; defensive `if (x == null)` sprawl.

### Don’t pass null
Avoid passing null as a parameter; use overloads, default objects, or optional parameters with clear semantics.
- **Smell to flag:** `foo(a, b, null, true)`; boolean flags that mean "sometimes null."

## Unit tests

### The Three Laws of TDD
1. **First:** You may not write production code until you have written a *failing* unit test.
2. **Second:** You may not write more of a unit test than is sufficient to fail; not compiling counts as failing.
3. **Third:** You may not write more production code than is sufficient to pass the current failing test.

### Keeping tests clean
Tests are first-class code: readable names, no duplication, clear arrange/act/assert structure, fast feedback.

### One assert per test (guideline)
Prefer one logical outcome per test so failures point to one behavior. Multiple asserts are acceptable when they assert *one* logical concept (same scenario).
- **Smell to flag:** one test validates login, profile load, and billing in sequence.

### Single concept per test
Do not test unrelated behaviors in one test method.
- **Smell to flag:** test name uses "and" twice for unrelated behaviors.

### FIRST
- **Fast** — slow suites do not get run.
- **Independent** — order-insensitive; no shared mutable fixtures.
- **Repeatable** — same result locally and in CI; no clock/network flakes without control.
- **Self-validating** — clear pass/fail; no manual inspection.
- **Timely** — written close to production code (often before, with TDD).

## Classes

### Classes should be small
Measure size by *responsibilities*, not only lines. Few public methods that align with one reason to change.
- **Smell to flag:** "God class" imports half the system; dozens of unrelated methods.

### Single responsibility
A class should have one reason to change—one axis of variation.
- **Smell to flag:** class edits whenever logging format, DB schema, *and* HTTP serialization change.

### Cohesion
Keep fields and methods that **change for the same reasons** together; a class should read like one idea, not a grab bag of unrelated lifecycles.
- **Smell to flag:** one cluster of methods only touches subset A of fields, another cluster only subset B, with no real collaboration between them.

## Systems

### Separate construction from use
Build object graphs in one place (composition root, factories); keep business logic free of `new` for every volatile dependency.
- **Smell to flag:** deep `new` chains inside domain logic; global singletons for everything.

### Separation of main
Push setup (config, wiring) to `main`/bootstrap; keep domain modules pure.
- **Smell to flag:** `main` logic mixed with algorithms.

### Factories
Encapsulate complex construction so callers depend on abstractions, not concrete assembly steps.
- **Smell to flag:** half-built objects passed around and "finished" later.

### Dependency injection
Inject dependencies through constructors or explicit setters/factories instead of hard-coded lookups.
- **Smell to flag:** `ServiceLocator.getInstance()` hidden inside domain code.

### Scaling up
Grow architecture incrementally; avoid big-bang frameworks until needs are clear.

### Cross-cutting concerns
Handle logging, security, transactions with coherent boundaries (middleware, aspects, decorators)—do not scatter copies.

### Test-driven system architecture
Use tests to drive modular boundaries; decouple so pieces can be tested in isolation.

### Optimize decision making
Defer irreversible decisions; keep options open with seams and interfaces.

### Use standards wisely, when they add demonstrable value
Adopt standards that reduce friction; reject ceremony that does not pay rent.

### Systems need a domain-specific language
Ubiquitous language in code (types, functions) should match how the business speaks about the problem.

## Emergence

Design quality **emerges** from simple rules applied repeatedly:

- **Run all tests** — green is the gate for refactor.
- **Refactoring** — continuous small improvements without changing behavior.
- **No duplications** — one authoritative expression of each rule.
- **Expressive** — names and structure reveal intent.
- **Minimal classes and methods** — delete dead code; resist speculative abstraction.

- **Smell to flag:** skipped tests "for now"; duplicated business rules in three layers.

## Concurrency

### JavaScript and TypeScript (async, event loop, workers)
Most TypeScript runs on a **single-threaded event loop**: “concurrency” is **interleaved** async work, not parallel CPU threads by default. Still avoid **shared mutable state** without an owner: module-level caches written from many call sites, **stale closures** over variables that change between scheduling and execution, and **races** when two async paths last-write-wins the same object. Prefer **immutable snapshots**, **narrow `async`/`await` boundaries**, **handled rejections** at module or request edges, and **cancellation** (`AbortSignal`) when the platform supports it. For **`Worker`** or **`SharedArrayBuffer`**, treat cross-thread memory as a **deliberate, documented** boundary—keep shared segments small and ordering explicit.
- **Smell to flag:** fire-and-forget promises with no `catch`; mutable module singleton updated from overlapping requests; `let` captured in async callbacks without fixing the intended value.

### Single responsibility
Separate concurrency policy from domain logic when complexity warrants it.
- **Smell to flag:** `synchronized` sprinkled through business methods without a story; mixing orchestration (`Promise.all`, retry) deep inside pure domain calculations.

### Limit the scope of data
Share less mutable state; prefer confinement to a single thread when possible.
- **Smell to flag:** global mutable maps touched from many call sites.

### Use copies of data
Prefer immutable snapshots or defensive copies at boundaries when sharing across threads.
- **Smell to flag:** passing mutable collections into thread pools without synchronization story.

### Threads should be as independent as possible
Design tasks that do not require fine-grained lock ordering across many objects.
- **Smell to flag:** fixed lock order undocumented; nested locks across subsystems.

### Know your library
On the JVM, use `java.util.concurrent`, structured concurrency, executors, and atomics where appropriate. In Node and browsers, know **Promise** composition, **queueMicrotask** vs macrotasks, framework schedulers (e.g. React concurrency), and when **`Atomics`**/`Worker` APIs are worth the complexity.

### Know your execution models
Understand whether you have actors, thread pools, event loops, or async/await—and what runs where.

### Keep synchronized sections small
Hold locks for the minimum work; avoid I/O and heavy work inside critical sections.
- **Smell to flag:** `synchronized` around network calls or DB access.

## First make it work, then make it right

Get correct behavior and feedback (tests) before polishing structure—then refactor safely under green tests.
- **Smell to flag:** endless renaming while behavior is still wrong or untested.

## Smells and heuristics

Use these as review prompts; prefer **small refactors** over debate.

| Heuristic | Rule | Smell to flag |
|-----------|------|----------------|
| Explanatory variables | Split complex expressions into well-named locals | Dense one-liners with three boolean operators |
| Understand the algorithm | Know complexity and invariants before micro-tuning | "magic fix" without invariant |
| Function names say what they do | Verb + object; honest about effects | Names contradict body |
| Prefer polymorphism | Replace growing type switches | New `case` every feature |
| Follow standard conventions | Match language & team style | one-off patterns |
| Replace magic numbers | Named constants with meaning | literals repeated for same rule |
| Be precise | Avoid vague words (`handle`, `stuff`) | ambiguous booleans |
| Structure over convention | Enforce invariants with types/APIs | "everyone knows" rules only in docs |
| Encapsulate conditionals | Named predicates | repeated compound `if` |
| Hidden temporal coupling | Order requirements explicit in API | step 2 must run before step 1 but no type encodes it |
| Don’t be arbitrary | Decisions have reasons | random constants without comment |
| Encapsulate boundary conditions | Off-by-one and limits in one place | `+1`/`length-1` scattered |
| One level of abstraction | Don't mix high and low level in one function | nested concerns in one block |
| Keep configurable data at high level | Defaults and policy at composition edge | magic URLs in deep logic |
| Avoid transitive navigation | Law of Demeter; tell, don't chain | `a.getB().getC().getD()` |
| Don’t inherit constants | Prefer static import or namespace | constant inheritance hierarchies |
| Names at appropriate abstraction | Speak at the layer you are in | HTTP verbs in domain entity names |
| Standard nomenclature | Use domain and pattern names | invented synonyms |
| Long names for long scopes | Short names only in tiny scopes | 2-letter globals |
| Avoid encodings | No type prefixes in names | `str`/`p`/`m_` clutter |

---

End of reference. Return to [SKILL.md](SKILL.md) for workflows and [EXAMPLES.md](EXAMPLES.md) for before/after patterns.
