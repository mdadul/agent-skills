# Builder Examples

## 1) Car and Manual Example

### Context
A `Car` is a complex object with seats, engine type, trip computer, and GPS. A corresponding `Manual` describes the same features. Both are assembled via the same construction steps but produce entirely different products.

### Design
- **Builder interface**: `reset()`, `setSeats(n)`, `setEngine(engine)`, `setTripComputer(bool)`, `setGPS(bool)`.
- **Concrete Builders**: `CarBuilder` → produces `Car`; `CarManualBuilder` → produces `Manual`.
- **Director**: `constructSportsCar(builder)` calls `reset`, `setSeats(2)`, `setEngine(SportEngine)`, `setTripComputer(true)`, `setGPS(true)`.
- **Client**: creates `CarBuilder`, passes to Director, retrieves `Car` from builder; repeats with `CarManualBuilder` to get `Manual`.

### Outcome
Director recipe is written once. Swapping the builder produces either a drivable car or a printed manual without changing the Director.

---

## 2) Telescoping Constructor Elimination (Pizza)

### Before
```java
new Pizza(12, true, false, true, false, true)  // which bool is which?
```
Six overloaded constructors, most parameters optional and easy to mix up.

### After (Fluent Builder)
```java
Pizza pizza = new Pizza.Builder(12)
    .cheese(true)
    .pepperoni(true)
    .mushrooms(true)
    .build();
```
- `Builder` is a static inner class of `Pizza`.
- `build()` validates required fields and calls the private `Pizza` constructor.
- `Pizza` is immutable; all fields are final.

### Outcome
Only required parameters are mandatory. Optional features are explicit and readable. The Pizza class remains immutable.

---

## 3) SQL Query Builder

### Context
Building SQL strings via string concatenation is error-prone and varies by query type (SELECT, INSERT, UPDATE).

### Design
- **Builder interface**: `from(table)`, `select(columns)`, `where(condition)`, `orderBy(column)`, `limit(n)`.
- **Concrete Builders**: `SelectQueryBuilder`, `InsertQueryBuilder`, `UpdateQueryBuilder` — each implements only the steps relevant to its query type and provides `build(): string`.
- **Director** (optional): `buildPaginatedQuery(builder, page, size)` encodes a reusable pagination recipe.

### Outcome
Query construction is centralized and testable. Adding a new query type (e.g., DELETE) requires only a new Concrete Builder.

---

## 4) Composite Tree Builder

### Context
Constructing a nested document (headings → sections → paragraphs) involves recursive structure that is hard to express via constructor chains.

### Design
- Builder exposes `addHeading(text)`, `openSection()`, `addParagraph(text)`, `closeSection()`.
- Internally maintains a node stack; step methods push/pop nodes.
- `build()` returns the root `Document` node.

### Note
Builder steps can call themselves recursively, making it natural for Composite trees where depth is not known at compile time.

---

## 5) Quick Evaluation Prompts
- "Refactor this constructor with eight parameters to use Builder."
- "Should I use Builder or just named parameters in Kotlin for this case?"
- "Show me a fluent Builder in TypeScript for an HTTP request object."
- "Do I need a Director here, or can the client call steps directly?"
- "Compare Builder vs Abstract Factory for creating themed UI component sets."
