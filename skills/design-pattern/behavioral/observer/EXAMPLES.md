# Observer Examples

## 1) Store Product Availability Notifications

### Context
Customers want updates only for products they subscribed to.

### Design
- Publisher: `Store` emits `productAvailable` events.
- Subscribers: `CustomerNotificationListener` implementations (email, SMS, push).
- Customers subscribe/unsubscribe dynamically by product interest.

### Outcome
No polling by customers and no spam to uninterested users.

## 2) Editor File Events

### Context
Editor should notify logging and alerting services when files open/save.

### Pattern Use
- `EventManager` manages subscribers by event type.
- `Editor` publishes `open` and `save` events.
- Subscribers include `LoggingListener` and `EmailAlertsListener`.

### Outcome
New reactions can be added without modifying editor core logic.

## 3) Order Lifecycle Event Bus

### Context
Order service state transitions must trigger inventory, billing, and analytics updates.

### Pattern Use
- Publisher emits `orderPlaced`, `orderPaid`, `orderCancelled`.
- Independent subscribers handle inventory reservation, invoice generation, metrics.
- Failure isolation prevents one subscriber from blocking others.

### Outcome
Domain reactions scale independently while preserving loose coupling.

## 4) Quick Evaluation Prompts
- "Evaluate whether this module needs Observer or Mediator."
- "Refactor this polling loop into subscription-based updates."
- "Design event contracts and subscriber lifecycle for this publisher."
- "Compare Observer vs Command for this notification workflow."
