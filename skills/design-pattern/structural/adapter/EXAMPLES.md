# Adapter Examples

## 1) XML Feed to JSON Analytics

### Context
Client uses XML stock feed; analytics SDK only accepts JSON payloads.

### Design
- Client interface expects `analyze(xmlFeed)`.
- Adapter wraps analytics SDK and converts XML to JSON before delegation.
- Adapter maps SDK response/errors back to client-facing model.

### Outcome
Analytics integration works without modifying legacy feed client code.

## 2) Square Peg in Round Hole

### Context
Client API accepts `RoundPeg`; service object is `SquarePeg`.

### Design
- `SquarePegAdapter` implements/extends round-peg contract.
- Adapter computes compatible radius from square width and delegates.

### Outcome
Client can evaluate fit through expected API without concrete-type coupling.

## 3) Payment Gateway Normalization

### Context
Different gateways expose different auth/capture/refund APIs and error codes.

### Pattern Use
- Define common `PaymentProvider` interface.
- Implement one adapter per gateway.
- Normalize requests, responses, and error taxonomy in adapters.

### Outcome
Checkout business logic remains provider-agnostic.

## 4) Quick Evaluation Prompts
- "Evaluate whether Adapter or Facade fits this integration boundary."
- "Design an object adapter for this third-party SDK mismatch."
- "Refactor direct provider calls behind adapters and show migration steps."
- "Create an adapter mismatch matrix for this legacy interface conversion."
