# Strategy Examples

## 1) Navigation Route Planning

### Context
Navigation app must support car, walking, bike, and transit routing.

### Design
- Context: `Navigator`.
- Strategy interface: `RouteStrategy.buildRoute(origin, destination)`.
- Concrete strategies: `RoadRoute`, `WalkingRoute`, `TransitRoute`, `BikeRoute`.
- UI/client selects strategy based on user mode.

### Outcome
New routing modes are added without bloating navigator core class.

## 2) Payment Fee Calculation

### Context
Checkout supports card, wallet, and bank transfer fee models.

### Pattern Use
- Strategy interface: `FeeStrategy.calculate(order)`.
- Context: `CheckoutPricingService` delegates fee computation.
- Selection based on payment method and region.

### Outcome
Pricing orchestration stays stable while fee rules evolve independently.

## 3) Compression Algorithm Selection

### Context
File service needs different compression algorithms by file type and SLA.

### Pattern Use
- Strategies: `ZipCompression`, `GzipCompression`, `NoCompression`.
- Context chooses strategy from policy config at runtime.

### Outcome
Compression behavior changes via configuration, not context rewrites.

## 4) Quick Evaluation Prompts
- "Evaluate if this large conditional should be refactored to Strategy."
- "Refactor these algorithm branches into context + strategy classes."
- "Compare Strategy vs State for this behavior-switching requirement."
- "Design strategy selection policy for this runtime configuration matrix."
