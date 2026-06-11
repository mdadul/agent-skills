# Factory Method Examples

## 1) Logistics Example

### Before
- Business logic directly constructs `Truck`.
- Adding `Ship` requires edits in many call sites.

### After
- `Transport` interface with `deliver()`.
- `RoadLogistics` creator returns `Truck`.
- `SeaLogistics` creator returns `Ship`.
- Client only calls creator flow and product interface methods.

## 2) Cross-Platform UI Example

### Context
Dialog flow should stay identical across Windows and Web, but button rendering differs.

### Design
- Product: `Button`.
- Concrete Products: `WindowsButton`, `HtmlButton`.
- Creator: `Dialog` with `createButton()` and shared `render()` flow.
- Concrete Creators: `WindowsDialog`, `WebDialog` override `createButton()`.

### Outcome
Shared dialog behavior remains stable while appearance/platform bindings vary per creator.

## 3) Resource Reuse Example

### Context
Constructing DB/network clients is expensive.

### Pattern Use
Factory method checks pool/cache first:
- return reusable object if available
- otherwise create, register, and return new object

### Note
Factory Method does not require always creating a new object; returning managed existing instances is valid.

## 4) Quick Evaluation Prompts
- "Refactor this class to Factory Method and explain each role."
- "Is Factory Method overkill for this creation flow?"
- "Show me Factory Method in TypeScript for payment providers."
- "Compare my Factory Method design with Abstract Factory for this case."
