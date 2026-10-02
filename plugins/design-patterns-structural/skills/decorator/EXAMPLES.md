# Decorator Examples

## 1) Notification System (Canonical Subclass Explosion)

### Before (subclass explosion)
```
Notifier
├── SMSNotifier
├── FacebookNotifier
├── SlackNotifier
├── SMSFacebookNotifier
├── SMSSlackNotifier
├── FacebookSlackNotifier
└── SMSFacebookSlackNotifier   ← 7 subclasses for 3 optional channels
```
N channels → 2ᴺ subclasses. Runtime channel selection is impossible.

### After (Decorator)
- **Component interface**: `Notifier` with `send(message)`.
- **Concrete Component**: `EmailNotifier` — base channel, always present.
- **Base Decorator**: `NotifierDecorator implements Notifier` — holds `wrappee`, delegates `send`.
- **Concrete Decorators**: `SMSDecorator`, `FacebookDecorator`, `SlackDecorator`.

```typescript
let notifier: Notifier = new EmailNotifier(emailList);
if (userWantsSMS)      notifier = new SMSDecorator(notifier, phoneList);
if (userWantsFacebook) notifier = new FacebookDecorator(notifier, fbToken);
if (userWantsSlack)    notifier = new SlackDecorator(notifier, slackWebhook);

notifier.send("Server is on fire!");
// → sends via every enabled channel in one call
```

Adding a new channel requires only one new decorator class and zero changes to existing code.

---

## 2) Data Source: Compression + Encryption Stack

### Context
Data written to a file must optionally be compressed, encrypted, or both. Order matters: compress before encrypt on write; decrypt before decompress on read.

### Design
- **Component interface**: `DataSource` — `writeData(data)`, `readData(): data`.
- **Concrete Component**: `FileDataSource`.
- **Base Decorator**: `DataSourceDecorator` — delegates both methods.
- **Concrete Decorators**: `CompressionDecorator`, `EncryptionDecorator`.

```
// Stack: Encryption > Compression > FileDataSource
source = new FileDataSource("salary.dat")
source = new CompressionDecorator(source)
source = new EncryptionDecorator(source)

source.writeData(data)
// → encrypt(compress(data)) → write to file

source.readData()
// → read from file → decrypt → decompress
```

### Note
Order is critical here. `EncryptionDecorator(CompressionDecorator(file))` produces correct results; reversing the stack does not.

---

## 3) HTTP Middleware Pipeline

### Context
An HTTP handler needs optional cross-cutting layers: logging, auth checking, response caching, rate limiting.

### Design
- **Component interface**: `Handler` — `handle(request): Response`.
- **Concrete Component**: `RouteHandler` — actual business logic.
- **Concrete Decorators**: `LoggingDecorator`, `AuthDecorator`, `CacheDecorator`, `RateLimitDecorator`.

```typescript
let handler: Handler = new RouteHandler(routes);
handler = new CacheDecorator(handler, cache);
handler = new AuthDecorator(handler, authService);
handler = new RateLimitDecorator(handler, limiter);
handler = new LoggingDecorator(handler, logger);

// Client calls the outermost decorator
response = handler.handle(request);
// → log → rate-check → auth-check → cache-or-route
```

### Note
Each decorator is independently testable. Swapping or removing a layer requires only changing the assembly code.

---

## 4) Beverage Pricing (Classic Java Example)

### Context
A coffee shop has a base beverage (Espresso, HouseBlend) with optional add-ons (milk, soy, mocha, whip). Cost and description accumulate across layers.

```java
Beverage b = new Espresso();          // $1.99 "Espresso"
b = new Mocha(b);                      // $2.29 "Espresso, Mocha"
b = new Mocha(b);                      // $2.59 "Espresso, Mocha, Mocha"
b = new Whip(b);                       // $2.89 "Espresso, Mocha, Mocha, Whip"

System.out.println(b.getDescription()); // "Espresso, Mocha, Mocha, Whip"
System.out.println(b.cost());           // 2.89
```

Each decorator adds to `cost()` and `getDescription()` by calling `super` and appending its contribution.

---

## 5) Pattern Comparison Table

| | Decorator | Proxy | Adapter | Chain of Responsibility |
|---|---|---|---|---|
| **Interface** | Same or extended | Same | Different | Same |
| **Wraps** | 1 component | 1 subject | 1 adaptee | 0–1 successor |
| **Purpose** | Add behavior | Control access/lifecycle | Convert interface | Route/stop requests |
| **Stack depth** | Arbitrary | Usually 1 | Usually 1 | Arbitrary |
| **Can stop flow** | No — must propagate | Conditionally | N/A | Yes |
| **Composed by** | Client | Proxy itself | Client | Client or chain builder |

---

## 6) Quick Evaluation Prompts
- "Refactor this notification subclass explosion to use Decorator."
- "Show me Decorator in TypeScript for an HTTP request pipeline."
- "What's the difference between Decorator and Proxy?"
- "Can I remove a specific decorator from the middle of the stack?"
- "When should I use Decorator vs Chain of Responsibility for middleware?"
