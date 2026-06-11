# Mediator Examples

## 1) Authentication Dialog Coordination

### Context
Login/register form has checkbox, text inputs, and buttons with interdependent behavior.

### Design
- Components emit events (`click`, `check`, `input`).
- `AuthenticationDialogMediator` handles visibility toggles and validation flow.
- Controls no longer call each other directly.

### Outcome
UI controls are reusable in other dialogs with different mediator rules.

## 2) Checkout Step Orchestration

### Context
Shipping, payment, coupon, and review components affect one another.

### Pattern Use
- `CheckoutMediator` receives field-change events.
- Updates dependent components (fees, eligibility, submit availability).
- Centralizes cross-step policies and error display routing.

### Outcome
Interaction logic changes are confined to mediator instead of scattered UI widgets.

## 3) Chat Room Coordination

### Context
Participants should not hold direct references to all other participants.

### Pattern Use
- `ChatRoomMediator` routes messages and enforces moderation/presence rules.
- Users notify mediator; mediator decides recipients and policy.

### Outcome
Participants remain decoupled while collaboration rules remain centralized.

## 4) Quick Evaluation Prompts
- "Evaluate whether this interaction graph should use Mediator or Observer."
- "Refactor direct component calls into mediator notifications."
- "Design a mediator split plan to avoid a god mediator in this module."
- "Compare Mediator vs Facade for this subsystem coordination problem."
