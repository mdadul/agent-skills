# State Examples

## 1) Document Workflow (Canonical Conditional Smell → State)

### Before
```typescript
class Document {
    state: "draft" | "moderation" | "published" = "draft";

    publish(user: User) {
        if (this.state === "draft") {
            this.state = "moderation";
        } else if (this.state === "moderation") {
            if (user.role === "admin") this.state = "published";
        } else if (this.state === "published") {
            // do nothing
        }
    }

    reject() {
        if (this.state === "moderation") this.state = "draft";
        // silent no-op otherwise
    }
}
```
Adding a new state (e.g., "archived") means editing `publish`, `reject`, and every other method.

### After
```
States:    Draft → Moderation → Published
Transitions:
  Draft.publish()       → Moderation
  Moderation.publish()  → Published (admin only)
  Moderation.reject()   → Draft
  Published.*           → no-op
```

```typescript
interface DocumentState {
    publish(doc: Document, user: User): void;
    reject(doc: Document): void;
}

class DraftState implements DocumentState {
    publish(doc: Document, _user: User) {
        doc.setState(new ModerationState());
    }
    reject(_doc: Document) { /* no-op in draft */ }
}

class ModerationState implements DocumentState {
    publish(doc: Document, user: User) {
        if (user.role === "admin") doc.setState(new PublishedState());
    }
    reject(doc: Document) {
        doc.setState(new DraftState());
    }
}

class PublishedState implements DocumentState {
    publish(_doc: Document, _user: User) { /* already published */ }
    reject(_doc: Document) { /* cannot reject published */ }
}

class Document {
    private state: DocumentState = new DraftState();  // starting state

    setState(s: DocumentState) { this.state = s; }
    publish(user: User)        { this.state.publish(this, user); }
    reject()                   { this.state.reject(this); }
}
```

Adding "Archived" state → one new class, wire two transitions. Zero existing code changes.

---

## 2) Audio Player (Canonical GoF Example)

### State Machine
```
          clickLock          clickLock
 Ready ──────────→ Locked ←──────────── Playing
   ↑    clickPlay                clickPlay ↓
   └──────────────────────────────────────┘
```

```typescript
abstract class PlayerState {
    constructor(protected player: AudioPlayer) {}
    abstract clickLock(): void;
    abstract clickPlay(): void;
    abstract clickNext(): void;
}

class ReadyState extends PlayerState {
    clickLock() { this.player.setState(new LockedState(this.player)); }
    clickPlay() {
        this.player.startPlayback();
        this.player.setState(new PlayingState(this.player));
    }
    clickNext() { this.player.nextSong(); }
}

class PlayingState extends PlayerState {
    clickLock() { this.player.setState(new LockedState(this.player)); }
    clickPlay() {
        this.player.stopPlayback();
        this.player.setState(new ReadyState(this.player));
    }
    clickNext() { this.player.fastForward(5); }
}

class LockedState extends PlayerState {
    clickLock() {
        const next = this.player.isPlaying()
            ? new PlayingState(this.player)
            : new ReadyState(this.player);
        this.player.setState(next);
    }
    clickPlay() { /* locked — no-op */ }
    clickNext()  { /* locked — no-op */ }
}
```

---

## 3) Traffic Light (Shared Flyweight States)

### Context
States are stateless — no per-context data. One instance of each state class can be shared across all traffic light objects.

```typescript
const RED    = new RedState();
const YELLOW = new YellowState();
const GREEN  = new GreenState();

class TrafficLight {
    private state: TrafficLightState = RED;

    tick() { this.state.tick(this); }
    setState(s: TrafficLightState) { this.state = s; }
}

class RedState implements TrafficLightState {
    tick(light: TrafficLight) {
        console.log("RED — stop");
        light.setState(GREEN);  // shared singleton, no allocation
    }
}
```

Since `RED`, `GREEN`, `YELLOW` are singletons with no instance data, they're safely shareable.

---

## 4) Order Lifecycle (E-Commerce)

### State Machine
```
Pending → Confirmed → Shipped → Delivered
    ↓          ↓
 Cancelled  Cancelled
```

Each state class only implements the transitions valid from that state; others throw or are no-ops.

```typescript
class PendingState implements OrderState {
    confirm(order: Order) { order.setState(new ConfirmedState()); }
    cancel(order: Order)  { order.setState(new CancelledState()); }
    ship(_order: Order)   { throw new Error("Cannot ship unconfirmed order"); }
}

class ConfirmedState implements OrderState {
    confirm(_order: Order) { /* already confirmed */ }
    cancel(order: Order)   { order.setState(new CancelledState()); }
    ship(order: Order)     { order.setState(new ShippedState()); }
}
```

Invalid transitions are explicit — no silent no-ops hiding bugs.

---

## 5) State vs Strategy

| | State | Strategy |
|---|---|---|
| **States/strategies know each other** | Yes — may trigger transitions | No — completely independent |
| **Transitions** | Explicit, driven by events | No transitions — client swaps |
| **Models** | Object lifecycle / FSM | Interchangeable algorithm |
| **Context delegates** | State-specific behavior | One algorithmic operation |
| **Adding new variant** | New state + wire transitions | New strategy class only |
| **Example** | Document workflow, player, traffic light | Sorting algorithm, payment processor |

Key rule: if "switching" from one to another is a meaningful domain event (lock/unlock, publish/reject), use State. If you're just selecting an algorithm at configuration time with no lifecycle meaning, use Strategy.

---

## 6) State Machine Diagram Template

```
[StateName]
  event1()  → [NextState]     (condition: optional)
  event2()  → [AnotherState]
  event3()  → no-op / error

[NextState]
  ...
```

Always draw this before writing code — it becomes the direct blueprint for which methods go in each class and which `setState` calls appear in each method.

---

## 7) Quick Evaluation Prompts
- "Refactor this if/switch state machine into the State pattern."
- "Should I use State or Strategy for this behavior toggle?"
- "Who should be responsible for triggering state transitions?"
- "Can I share state objects across multiple context instances?"
- "Show me a State pattern implementation in TypeScript for an order lifecycle."
