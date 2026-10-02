# Pragmatic Programmer — Examples

Short **before → after** pairs in **TypeScript**. Use with [SKILL.md](SKILL.md) and [REFERENCE.md](REFERENCE.md).

---

## DRY

### 1. Impatient duplication — copy-paste with tiny edits

**Before**
```typescript
function validateOrderItem(item: OrderItem) {
  if (item.quantity <= 0) throw new Error("Quantity must be positive");
  if (item.price < 0) throw new Error("Price cannot be negative");
}

function validateCartItem(item: CartItem) {
  if (item.quantity <= 0) throw new Error("Quantity must be positive");
  if (item.price < 0) throw new Error("Price cannot be negative");
}
```

**After**
```typescript
function validateLineItem(item: { quantity: number; price: number }) {
  if (item.quantity <= 0) throw new Error("Quantity must be positive");
  if (item.price < 0) throw new Error("Price cannot be negative");
}
```
One function owns the rule. Both `OrderItem` and `CartItem` satisfy the structural type — no divergence risk.

---

### 2. Inadvertent duplication — same rule, different names

**Before**
```typescript
// In domain/order.ts
const MAX_LINE_ITEMS = 50;
if (order.lines.length > MAX_LINE_ITEMS) throw new Error("Too many items");

// In api/cart-controller.ts
if (cart.items.length > 50) return res.status(400).json({ error: "Cart too large" });
```

**After**
```typescript
// In domain/constants.ts
export const MAX_LINE_ITEMS = 50;

// In domain/order.ts
import { MAX_LINE_ITEMS } from "../constants";
if (order.lines.length > MAX_LINE_ITEMS) throw new Error("Too many items");

// In api/cart-controller.ts
import { MAX_LINE_ITEMS } from "../domain/constants";
if (cart.items.length > MAX_LINE_ITEMS) return res.status(400).json({ error: "Cart too large" });
```
The limit is now a single authoritative fact. Change it once; both sites follow.

---

### 3. Imposed duplication — type and runtime validator

**Before**
```typescript
// TypeScript type and Zod schema maintained separately — must stay in sync manually
interface UserInput {
  email: string;
  age: number;
  role: "admin" | "member";
}

const userInputSchema = z.object({
  email: z.string().email(),
  age: z.number().int().min(0),
  role: z.enum(["admin", "member"]),
});
```

**After**
```typescript
// Single source: derive the TypeScript type from the schema
const userInputSchema = z.object({
  email: z.string().email(),
  age: z.number().int().min(0),
  role: z.enum(["admin", "member"]),
});

type UserInput = z.infer<typeof userInputSchema>;
```
Adding a field to the schema automatically updates the type — one change, one place.

---

### 4. DRY in tests — test hard-codes a production constant

**Before**
```typescript
// production code
const SESSION_TIMEOUT_MS = 30 * 60 * 1000; // 30 minutes

// test
it("expires session after 30 minutes", () => {
  advanceTime(30 * 60 * 1000); // magic number, must be kept in sync manually
  expect(session.isExpired()).toBe(true);
});
```

**After**
```typescript
import { SESSION_TIMEOUT_MS } from "../auth/constants";

it("expires session after the configured timeout", () => {
  advanceTime(SESSION_TIMEOUT_MS);
  expect(session.isExpired()).toBe(true);
});
```
The test now references the same constant that production code uses — no silent divergence.

---

### 5. Interdeveloper duplication — three `formatCurrency` helpers

**Before**
```typescript
// checkout/utils.ts
function formatCurrency(cents: number) { return `$${(cents / 100).toFixed(2)}`; }

// invoice/helpers.ts
function toDollarString(amount: number) { return "$" + (amount / 100).toFixed(2); }

// reporting/format.ts
const asMoney = (v: number) => `$${(v / 100).toFixed(2)}`;
```

**After**
```typescript
// shared/money.ts
export function formatCents(cents: number): string {
  return `$${(cents / 100).toFixed(2)}`;
}
```
All three modules import from one place. Business logic for currency display lives once.

---

## Orthogonality

### 6. Decoupling — Law of Demeter violation

**Before**
```typescript
function shipOrder(order: Order) {
  const city = order.getCustomer().getAddress().getCity();
  courier.dispatchTo(city);
}
```
Three levels of internal structure exposed to `shipOrder`. Changing `Customer` or `Address` breaks this.

**After**
```typescript
// Order exposes only what callers need
class Order {
  shippingCity(): string {
    return this.customer.shippingCity();
  }
}

function shipOrder(order: Order) {
  courier.dispatchTo(order.shippingCity());
}
```
`shipOrder` is now orthogonal to `Customer` and `Address` internals.

---

### 7. Orthogonality — infrastructure leaking into domain

**Before**
```typescript
class UserService {
  async find(id: string, res: Response) {
    const user = await db.query(`SELECT * FROM users WHERE id = $1`, [id]);
    if (!user) return res.status(404).json({ error: "Not found" });
    return res.json(user);
  }
}
```
HTTP `Response` is an infrastructure concern. This domain service now knows about HTTP.

**After**
```typescript
// domain/user-service.ts
class UserService {
  async find(id: string): Promise<User | undefined> {
    return db.users.findById(id);
  }
}

// api/user-controller.ts
async function getUser(req: Request, res: Response) {
  const user = await userService.find(req.params.id);
  if (!user) return res.status(404).json({ error: "Not found" });
  res.json(user);
}
```
Domain and HTTP are orthogonal — changing one does not affect the other.

---

### 8. Orthogonality — global mutable state

**Before**
```typescript
// auth.ts
export let currentUser: User | null = null;

// middleware.ts
import { currentUser } from "./auth";
currentUser = await resolveUser(req.headers.authorization);

// billing.ts
import { currentUser } from "./auth";
if (currentUser?.plan === "free") throw new Error("Upgrade required");
```
`billing.ts` and `middleware.ts` are secretly coupled through `currentUser`. Order of writes matters.

**After**
```typescript
// auth.ts
export async function resolveUser(token: string): Promise<User> { ... }

// middleware.ts
async function authMiddleware(req: Request, res: Response, next: NextFunction) {
  req.user = await resolveUser(req.headers.authorization ?? "");
  next();
}

// billing.ts
function assertPaidPlan(user: User) {
  if (user.plan === "free") throw new Error("Upgrade required");
}

// controller.ts
router.get("/feature", authMiddleware, (req, res) => {
  assertPaidPlan(req.user);
  ...
});
```
`billing.ts` receives the user it needs — no hidden global dependency.

---

### 9. Similar functions — extract the differing behavior

**Before**
```typescript
async function sendWelcomeEmail(user: User) {
  const transport = createTransport(smtpConfig);
  const html = renderTemplate("welcome", { name: user.name });
  await transport.sendMail({ to: user.email, subject: "Welcome!", html });
}

async function sendPasswordResetEmail(user: User, token: string) {
  const transport = createTransport(smtpConfig);
  const html = renderTemplate("password-reset", { name: user.name, token });
  await transport.sendMail({ to: user.email, subject: "Reset your password", html });
}
```
SMTP setup and dispatch duplicated. Adding BCC or retry logic requires two edits.

**After**
```typescript
async function sendEmail(to: string, subject: string, template: string, data: object) {
  const transport = createTransport(smtpConfig);
  const html = renderTemplate(template, data);
  await transport.sendMail({ to, subject, html });
}

async function sendWelcomeEmail(user: User) {
  await sendEmail(user.email, "Welcome!", "welcome", { name: user.name });
}

async function sendPasswordResetEmail(user: User, token: string) {
  await sendEmail(user.email, "Reset your password", "password-reset", { name: user.name, token });
}
```
Infrastructure concern lives once. Both callers remain clean and distinct.

---

### 10. Combined DRY + Orthogonality — parallel switch trees

**Before**
```typescript
// Two switch trees on the same `PaymentMethod` enum — must be updated together
function getPaymentLabel(method: PaymentMethod): string {
  switch (method) {
    case "card": return "Credit Card";
    case "bank": return "Bank Transfer";
    case "crypto": return "Cryptocurrency";
  }
}

function getPaymentIcon(method: PaymentMethod): string {
  switch (method) {
    case "card": return "💳";
    case "bank": return "🏦";
    case "crypto": return "🪙";
  }
}
```
Adding a new payment method requires editing two places. Each switch is a duplicate of the other's structure.

**After**
```typescript
const PAYMENT_METHOD_META: Record<PaymentMethod, { label: string; icon: string }> = {
  card:   { label: "Credit Card",    icon: "💳" },
  bank:   { label: "Bank Transfer",  icon: "🏦" },
  crypto: { label: "Cryptocurrency", icon: "🪙" },
};

function getPaymentLabel(method: PaymentMethod): string {
  return PAYMENT_METHOD_META[method].label;
}

function getPaymentIcon(method: PaymentMethod): string {
  return PAYMENT_METHOD_META[method].icon;
}
```
Adding a new method is one edit in one place. TypeScript's `Record` enforces completeness at compile time.
