# Adapter Reference

## Intent
Adapter makes incompatible interfaces collaborate by translating calls, data, and expectations between client and service.

## Problem Signal
- Useful service exists but interface is incompatible with current client contract.
- Data formats differ (for example XML vs JSON, metric vs imperial).
- Integration code is duplicated across clients with one-off conversions.
- Service cannot be changed directly (third-party, legacy, risk constraints).

## Solution Shape
1. Keep/define a stable client interface.
2. Wrap incompatible service in an adapter implementing that client interface.
3. Translate request/response models and error semantics.
4. Keep client unaware of concrete service API.
5. Add one adapter per incompatible service variant.

## Roles
- Client: Existing business logic expecting a specific interface.
- Client Interface: Contract used by client.
- Service (Adaptee): Existing incompatible API.
- Adapter: Translator implementing client interface and delegating to service.

## Adapter Styles
- Object adapter (composition): preferred in most languages.
- Class adapter (multiple inheritance): niche; language-limited.

## Applicability
Use when:
- Interface mismatch blocks integration.
- Service modification is impractical or impossible.
- You need to normalize multiple providers under one contract.

Avoid when:
- No interface mismatch exists (Decorator/Proxy may fit better).
- Service can be safely redesigned with lower long-term complexity.

## Refactor Recipe
1. Identify mismatches: method names, payload schema, units, sync model, errors.
2. Define client-facing interface and test expectations.
3. Implement adapter with wrapped service dependency.
4. Move all conversion logic into adapter.
5. Replace direct service calls with client-interface usage.
6. Add provider-specific adapters as needed.

## Validation Checklist
- Client behavior unchanged after integration.
- Conversion logic is reversible or intentionally lossy with documented rules.
- Service errors map to client-facing error model with context.
- Performance impact from conversion is acceptable.
- New service provider can be integrated via new adapter only.

## Pros
- Isolates translation complexity from business logic.
- Improves extensibility through boundary abstractions.
- Enables reuse of legacy/third-party services safely.

## Cons
- Adds indirection and additional classes.
- Can hide design debt if used where direct redesign is better.

## Relationship Notes
- Adapter changes interface; Decorator keeps/extents interface behavior.
- Facade simplifies subsystem access; Adapter translates one interface to another.
- Proxy keeps same interface and controls access; Adapter changes interface.
- Bridge is usually upfront architecture; Adapter is often retrofit integration.
