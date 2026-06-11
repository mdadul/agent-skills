# Chain of Responsibility Examples

## 1) Order System Validation Pipeline (Canonical Problem)

### Before (monolithic)
```typescript
function handleOrder(request: OrderRequest) {
    // Auth check
    if (!request.credentials) throw new Error("Unauthenticated");
    // Sanitize
    if (!isClean(request.data)) throw new Error("Invalid data");
    // Rate limit
    if (isRateLimited(request.ip)) throw new Error("Too many requests");
    // Cache check
    const cached = cache.get(request.id);
    if (cached) return cached;
    // Process
    return orderService.process(request);
}
```
Adding a new check means editing this function. Reusing just the auth check elsewhere means duplicating it.

### After (pass-through pipeline variant)
- **Handler interface**: `handle(request: OrderRequest): OrderResponse`.
- **Concrete Handlers**: `AuthHandler`, `SanitizationHandler`, `RateLimitHandler`, `CacheHandler`, `OrderProcessorHandler`.

```typescript
abstract class OrderHandler {
    private next: OrderHandler | null = null;

    setNext(handler: OrderHandler): OrderHandler {
        this.next = handler;
        return handler;  // enables fluent chaining
    }

    protected forward(request: OrderRequest): OrderResponse {
        if (this.next) return this.next.handle(request);
        throw new Error("No handler processed the request");
    }

    abstract handle(request: OrderRequest): OrderResponse;
}

class AuthHandler extends OrderHandler {
    handle(request: OrderRequest): OrderResponse {
        if (!request.credentials) throw new UnauthorizedError();
        return this.forward(request);  // always forwards on success
    }
}
```

### Chain Assembly
```typescript
const auth = new AuthHandler();
const sanitize = new SanitizationHandler();
const rateLimit = new RateLimitHandler();
const cache = new CacheHandler();
const processor = new OrderProcessorHandler();

auth.setNext(sanitize).setNext(rateLimit).setNext(cache).setNext(processor);

const response = auth.handle(request);
```

Adding a new check → one new handler class, one line in assembly. Reusing `AuthHandler` in another pipeline → just include it in that chain.

---

## 2) GUI Event Bubbling (Stop-on-First + Composite Tree)

### Context
Pressing F1 on a UI element triggers contextual help. The element tries to show help; if it can't, it passes to its container, up to the root dialog.

### Natural CoR from Composite tree
```
Dialog (wikiPageURL → opens browser)
└── Panel (modalHelpText → shows modal)
    ├── OKButton (tooltipText → shows tooltip)
    └── CancelButton (no help text → forwards to Panel)
```

`CancelButton.showHelp()` → no tooltip → forward to `Panel` → has modal text → **stops here**.
`OKButton.showHelp()` → has tooltip → **stops here**.

### Design
- **Handler interface**: `showHelp()`.
- **Base Handler** (`Component`): holds `container: Container`; default `showHelp()` forwards to container.
- **Concrete Handlers**: `Button`, `Panel`, `Dialog` — each overrides only if it can provide help.

```typescript
abstract class Component {
    container?: Container;

    showHelp(): void {
        if (this.container) this.container.showHelp();
        // else: silently unhandled — root with no help configured
    }
}

class Panel extends Container {
    modalHelpText?: string;

    showHelp(): void {
        if (this.modalHelpText) showModal(this.modalHelpText);
        else super.showHelp();  // forward to parent container
    }
}
```

---

## 3) HTTP Middleware (Express-style Pass-Through)

### Context
An Express-like server needs: request logging, JWT auth, request body validation, response compression.

```typescript
// Each middleware: process → call next() → done (pass-through)
app.use(requestLogger);    // always logs, always calls next()
app.use(jwtAuth);          // validates token; 401 if invalid, next() if valid
app.use(bodyValidator);    // validates schema; 400 if invalid, next() if valid
app.use(compression);      // compresses response; always calls next()
app.use(routeHandler);     // terminal handler — no next() call
```

This IS the pass-through CoR variant. Each middleware is a Concrete Handler. The framework's `app.use()` is the chain assembly API.

---

## 4) Tech Support Escalation (Stop-on-First)

### Context
A support ticket is routed: frontline agent → specialist → engineering team. Each level handles what it can; escalates the rest.

```typescript
class FrontlineAgent extends SupportHandler {
    handle(ticket: Ticket): Resolution {
        if (ticket.priority === "LOW") return this.resolveFromFAQ(ticket);
        return this.forward(ticket);
    }
}

class Specialist extends SupportHandler {
    handle(ticket: Ticket): Resolution {
        if (ticket.priority === "MEDIUM") return this.diagnose(ticket);
        return this.forward(ticket);
    }
}

class Engineer extends SupportHandler {
    handle(ticket: Ticket): Resolution {
        return this.deepInvestigate(ticket);  // terminal — no forward
    }
}

// Chain assembly
frontline.setNext(specialist).setNext(engineer);
```

---

## 5) CoR vs Related Patterns

| | CoR | Decorator | Mediator | Observer |
|---|---|---|---|---|
| **Can stop propagation** | Yes | No | N/A | No |
| **Topology** | Linear chain | Linear wrap stack | Star (hub) | Fan-out broadcast |
| **Handler count** | One (stop-on-first) or all (pass-through) | All always | All via mediator | All subscribers |
| **Request source** | Any handler in chain | Outermost decorator | Sender → mediator | Publisher |
| **Handler knows next** | Yes (via interface) | Yes (via interface) | No | No |
| **Primary purpose** | Route/handle requests | Enrich behavior | Decouple many-to-many | Notify on events |

---

## 6) Unhandled Request Strategies

| Strategy | When to use |
|---|---|
| Throw exception at end of chain | Request must always be handled; unhandled = bug |
| Return null / empty result | Optional handling; caller checks for null |
| Default terminal handler | Catches all unmatched requests with a fallback response |
| Log and discard | Fire-and-forget; unhandled is acceptable |

---

## 7) Quick Evaluation Prompts
- "Refactor this growing if-else validation chain to use CoR."
- "Should I use pass-through pipeline or stop-on-first for my middleware?"
- "What's the difference between CoR and Decorator?"
- "Show me a fluent chain builder API in TypeScript."
- "How do I handle requests that no handler in the chain matches?"
