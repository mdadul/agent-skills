---
name: proxy
description: Teach and apply the Proxy pattern to substitute a service with an interchangeable wrapper that controls access while preserving the same interface. Use when users ask about Proxy, lazy initialization, access control, remote service wrappers, request logging, caching, or comparing Proxy with Decorator, Adapter, or Facade.
---

# Proxy

## Purpose
Help the user decide whether Proxy is appropriate, then design and implement same-interface wrappers that manage service access, lifecycle, and cross-cutting control concerns.

## When To Use
- User asks to explain Proxy pattern.
- Service is expensive and should be initialized lazily.
- Access to service requires authorization or policy checks.
- Service is remote and network details should be hidden from client code.
- Repeated requests benefit from caching/logging at the boundary.
- Client must remain unchanged while service access behavior is controlled.

## Inputs
- Service interface and current service implementation.
- Access concern type (virtual, protection, remote, caching, logging, smart reference).
- Lifecycle needs (creation timing, reuse, eviction, teardown).
- Error/retry/timeout expectations and observability requirements.
- Thread-safety and consistency requirements under concurrent access.

## Workflow
1. Clarify the goal.
   Confirm whether the user needs concept explanation, architecture review, or implementation/refactor.
2. Check applicability.
   Use Proxy when interface must remain identical while access behavior changes.
3. Define service contract.
   Ensure proxy and real service are interchangeable via one shared interface.
4. Choose proxy flavor.
   Select virtual, protection, remote, caching, logging, or smart-reference behavior.
5. Implement proxy wrapper.
   Add reference to real service and implement service interface methods.
6. Add control logic.
   Execute checks/caching/lazy initialization/telemetry before and after delegation.
7. Centralize service lifecycle.
   Let proxy create/manage service when appropriate, or accept service injection explicitly.
8. Migrate clients safely.
   Replace direct service wiring with proxy instances without changing client call sites.
9. Validate transparency.
   Verify functional parity, policy enforcement, and acceptable latency overhead.

## Decision Branches
- If interface translation is needed between incompatible APIs:
  Prefer Adapter.
- If goal is adding optional behaviors by stacking wrappers:
  Prefer Decorator.
- If goal is simplifying many subsystem classes behind a new API:
  Prefer Facade.
- If only object construction must vary:
  Prefer Factory Method or Abstract Factory.
- If proxy accumulates many unrelated responsibilities:
  Split concerns into dedicated proxies or support components.

## Output Contract
When responding, provide:
1. A suitability verdict (Proxy or better alternative).
2. Proxy flavor recommendation and rationale.
3. Role mapping (Client, Service Interface, Real Service, Proxy).
4. Incremental migration plan from direct service usage to proxy wiring.
5. Minimal code sketch in the user's language with one pre/post delegation concern.
6. Validation checklist proving transparency and control effectiveness.

## Quality Checks
- Proxy implements exactly the same service interface as real service.
- Client code depends on interface and remains unchanged after swap.
- Proxy concern is explicit and cohesive (auth, cache, remote, etc.).
- Delegation preserves service semantics and error contracts.
- Lifecycle strategy is defined (lazy create, reuse, disposal, invalidation).
- Performance overhead is measured and acceptable for critical paths.

## Common Mistakes To Prevent
- Changing interface in proxy and breaking substitutability.
- Mixing unrelated concerns into one proxy and creating a god wrapper.
- Caching non-idempotent operations without correctness rules.
- Silent error swallowing while adding logging/telemetry.
- Confusing Proxy with Decorator (enhancement) or Adapter (translation).

## References
- Detailed guide: [REFERENCE.md](./REFERENCE.md)
- Worked examples: [EXAMPLES.md](./EXAMPLES.md)
