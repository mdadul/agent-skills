# Facade Examples

## 1) Video Conversion Framework

### Context
Client currently uses many codec/reader/mixer classes directly.

### Design
- Facade: `VideoConverter` with `convert(filename, format)`.
- Facade handles codec selection, bitrate conversion, audio fixups, and file output.

### Outcome
Client integration shrinks to one high-level call and remains stable if framework internals change.

## 2) E-commerce Checkout Flow

### Context
Checkout client coordinates cart validation, pricing, payment, inventory, and shipping modules.

### Pattern Use
- Facade: `CheckoutService.placeOrder(request)`.
- Internally orchestrates subsystems and transactional order.
- Returns domain-level result model.

### Outcome
Controller/business layer is decoupled from subsystem choreography and ordering details.

## 3) Multi-Layer Data Platform API

### Context
Analytics subsystem has separate ingestion, transformation, and export layers.

### Pattern Use
- `IngestionFacade`, `TransformationFacade`, `ExportFacade` as focused entry points.
- Layer interaction occurs through facades rather than direct cross-layer calls.

### Outcome
Subsystem boundaries are explicit and easier to maintain or version.

## 4) Quick Evaluation Prompts
- "Evaluate if this integration should use Facade or Adapter and explain why."
- "Refactor this subsystem-heavy service into a task-focused facade API."
- "Design additional facades to split this oversized facade safely."
- "Compare Facade vs Mediator for this coordination-heavy module."
