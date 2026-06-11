# Proxy Reference

## Intent
Proxy provides a stand-in object with the same interface as a service, controlling access before or after delegation.

## Problem Signal
- Heavy service is eagerly created but used infrequently.
- Security/policy checks are duplicated across client code.
- Remote calls/network complexity leaks into business logic.
- Request logging/caching concerns are scattered around callers.

## Solution Shape
1. Share one service interface between real service and proxy.
2. Implement proxy methods with pre/post control behavior.
3. Delegate core work to real service.
4. Manage service lifecycle in proxy where appropriate.
5. Keep client usage unchanged by wiring proxy through same interface.

## Roles
- Service Interface: Common API contract.
- Real Service: Core business implementation.
- Proxy: Access-control wrapper and lifecycle manager.
- Client: Uses interface without knowing if service or proxy is wired.

## Common Proxy Flavors
- Virtual Proxy: Lazy initialization of expensive service.
- Protection Proxy: Authorization/policy checks.
- Remote Proxy: Network boundary wrapper for remote calls.
- Caching Proxy: Memoization and invalidation lifecycle.
- Logging Proxy: Request/audit tracing around service calls.
- Smart Reference: Reference counting/resource release when unused.

## Applicability
Use when:
- Interface must stay unchanged while access behavior changes.
- Service creation/usage requires centralized control policies.

Avoid when:
- Interface incompatibility is the main issue (Adapter).
- You need a simplified new API over subsystem classes (Facade).

## Refactor Recipe
1. Confirm interface-based client wiring is in place.
2. Implement proxy with matching interface.
3. Add one focused control concern first.
4. Delegate to real service and preserve contract.
5. Switch wiring to proxy and run parity tests.
6. Add additional proxy concerns only if cohesive.

## Validation Checklist
- Proxy can replace real service without client code changes.
- Authorization/cache/lazy behavior works as intended.
- Error semantics are preserved (or intentionally mapped with docs).
- Performance overhead is within acceptable budget.
- Concurrent access does not corrupt proxy state.

## Pros
- Centralizes service-control concerns.
- Keeps clients unaware of lifecycle/access complexity.
- Supports Open/Closed extension via new proxy types.

## Cons
- Adds indirection and potential latency.
- Easy to over-complicate with too many responsibilities.

## Relationship Notes
- Adapter changes interface; Proxy keeps the same interface.
- Decorator adds behavior composition; Proxy controls access/lifecycle.
- Facade simplifies subsystem API; Proxy stands in for one service contract.
