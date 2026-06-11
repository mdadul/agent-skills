# Bridge Examples

## 1) Shape × Color (Canonical Hierarchy Explosion)

### Before (inheritance, explodes)
```
Shape
├── Circle
│   ├── RedCircle
│   └── BlueCircle
└── Square
    ├── RedSquare
    └── BlueSquare
```
Adding a Triangle needs 2 new classes. Adding Green needs 3 new classes. M shapes × N colors = M×N classes.

### After (Bridge)
- **Implementation interface**: `Color` with `applyFill(): string`.
- **Concrete Implementations**: `Red`, `Blue`.
- **Abstraction**: `Shape` holds a `Color` reference; `draw()` delegates fill to `color.applyFill()`.
- **Refined Abstractions**: `Circle`, `Square` — each adds its own shape-drawing logic.

```
Shape (holds Color)
├── Circle
└── Square

Color (interface)
├── Red
└── Blue
```

Adding Triangle → 1 new Abstraction class, 0 Color changes.
Adding Green → 1 new Color class, 0 Shape changes.

---

## 2) Remote Control × Device (Canonical GoF Example)

### Design
- **Implementation interface**: `Device` — `enable()`, `disable()`, `isEnabled()`, `getVolume()`, `setVolume(n)`, `getChannel()`, `setChannel(n)`.
- **Concrete Implementations**: `Tv`, `Radio`.
- **Abstraction**: `RemoteControl` — holds a `Device`; implements `togglePower()`, `volumeUp/Down()`, `channelUp/Down()`.
- **Refined Abstraction**: `AdvancedRemoteControl extends RemoteControl` — adds `mute()`.

### Client
```typescript
const tv = new Tv();
const remote = new RemoteControl(tv);
remote.togglePower();

const radio = new Radio();
const advanced = new AdvancedRemoteControl(radio);
advanced.mute();
```

### Outcome
Adding a `SmartTV` device requires only a new `Device` implementation. Adding a `VoiceRemote` requires only a new Abstraction subclass. Neither change affects the other hierarchy.

---

## 3) Cross-Platform GUI × OS API

### Context
A desktop app renders a GUI (forms, buttons, dialogs) that must run on Windows, Linux, and macOS. Naive approach: `WindowsForm`, `LinuxForm`, `MacForm`, `WindowsDialog`, `LinuxDialog`, `MacDialog`…

### Bridge Design
- **Implementation interface**: `PlatformAPI` — `drawButton(label)`, `drawTextInput(placeholder)`, `showAlert(message)`.
- **Concrete Implementations**: `WindowsAPI`, `LinuxAPI`, `MacAPI`.
- **Abstraction**: `View` — holds a `PlatformAPI`; defines `render()` using API primitives.
- **Refined Abstractions**: `FormView`, `DialogView`, `DashboardView`.

### Outcome
Adding a new platform requires one new `PlatformAPI` class. Adding a new view type requires one new `View` subclass. 3 platforms × N views = 3 + N classes instead of 3N.

---

## 4) Notification × Delivery Channel

### Context
A system sends notifications (order confirmation, password reset, system alert) via multiple channels (email, SMS, push). Naively: `EmailOrderConfirmation`, `SmsOrderConfirmation`, `PushOrderConfirmation`, … 3 notifications × 3 channels = 9 classes.

### Bridge Design
- **Implementation interface**: `NotificationChannel` — `send(recipient, subject, body)`.
- **Concrete Implementations**: `EmailChannel`, `SmsChannel`, `PushChannel`.
- **Abstraction**: `Notification` — holds a `NotificationChannel`; defines `notify(user)`.
- **Refined Abstractions**: `OrderConfirmation`, `PasswordReset`, `SystemAlert` — each formats subject/body and calls `channel.send(...)`.

### Outcome
Adding Slack channel → 1 new class. Adding a new notification type → 1 new class. Runtime channel swap (e.g., user prefers SMS over email) is trivial: reassign the channel reference.

---

## 5) Bridge vs Adapter vs Strategy (Quick Comparison)

| | Bridge | Adapter | Strategy |
|---|---|---|---|
| **When applied** | Up-front design | After the fact | Either |
| **Goal** | Decouple two independent hierarchies | Make incompatible interfaces work together | Swap interchangeable algorithms |
| **Structure** | Composition (Abstraction → Implementation) | Wraps an incompatible object | Composition (Context → Strategy) |
| **Hierarchy count** | Two independent hierarchies | One adapter per adaptee | One strategy per algorithm |
| **Identifies** | Two orthogonal dimensions | Interface mismatch | Variable behavior in one dimension |

---

## 6) Quick Evaluation Prompts
- "My class hierarchy is exploding — is Bridge the right fix?"
- "Show me Bridge for a rendering engine that supports OpenGL and Vulkan."
- "What's the difference between Bridge and Strategy?"
- "Refactor this notification system to use Bridge."
- "When should I use Abstract Factory with Bridge?"
