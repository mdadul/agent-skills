# Facade Reference

## Intent
Facade offers a simplified, stable interface to a complex subsystem, reducing client coupling and orchestration burden.

## Problem Signal
- Clients instantiate and coordinate many subsystem classes directly.
- Repeated setup and call ordering logic appears across modules.
- Upgrading/replacing framework internals causes wide client breakage.
- Subsystem complexity leaks into business logic and tests.

## Solution Shape
1. Define a task-oriented facade API around common client scenarios.
2. Encapsulate subsystem initialization and call sequencing.
3. Translate low-level subsystem outputs/errors into client-facing results.
4. Keep subsystem classes unaware of facade and free to evolve internally.
5. Add additional focused facades if one facade grows too broad.

## Roles
- Client: Uses simplified facade API.
- Facade: Coordinates subsystem operations and lifecycle.
- Additional Facade (optional): Narrower entry points per domain/layer.
- Subsystem: Existing collaborating classes and services.

## Applicability
Use when:
- Complex subsystem usage overwhelms client code.
- You need a stable boundary while internals evolve.
- You want layered subsystem entry points to reduce coupling.

Avoid when:
- The issue is interface mismatch between two classes (Adapter fits better).
- You only need access/lifecycle control with unchanged interface (Proxy fits better).

## Refactor Recipe
1. Identify frequent client workflows and dependency orchestration hotspots.
2. Design minimal facade methods for those workflows.
3. Move subsystem wiring/ordering/lifecycle into facade.
4. Migrate clients incrementally to facade methods.
5. Remove direct subsystem access from client modules.
6. Split facade if responsibilities become unrelated.

## Validation Checklist
- Client modules call facade API instead of subsystem internals.
- Boilerplate setup and ordering logic is eliminated from clients.
- Facade methods are cohesive and use-case driven.
- Subsystem changes are localized to facade implementation.
- Facade does not become a dumping ground for unrelated operations.

## Pros
- Reduces cognitive load and coupling for clients.
- Localizes integration complexity and upgrade impact.
- Supports clearer layering boundaries.

## Cons
- Risk of god object if scope is not controlled.
- Possible capability loss if facade API is too narrow.

## Relationship Notes
- Adapter translates one interface to another; Facade simplifies a subsystem API.
- Proxy preserves same interface while controlling access.
- Mediator coordinates peer communication patterns, not just subsystem entry points.
- Abstract Factory can complement Facade when construction needs separate abstraction.
