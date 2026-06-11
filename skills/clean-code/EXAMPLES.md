# Clean Code — Examples

Short **before → after** pairs in **TypeScript** only. Use with [SKILL.md](SKILL.md) and [REFERENCE.md](REFERENCE.md).

## 1. Intention-revealing name

**Before**
```typescript
const d = Date.now() - file.modifiedAt;
if (d > thresholdMs) archive(file);
```

**After**
```typescript
const millisecondsSinceModified = Date.now() - file.modifiedAt;
if (millisecondsSinceModified > thresholdMs) archive(file);
```

## 2. Avoid disinformation

**Before**
```typescript
// Name says "list" but the value is a Set
const accountList = new Set<Account>();
```

**After**
```typescript
const accounts = new Set<Account>();
```

## 3. Searchable name (magic literal)

**Before**
```typescript
if (employee.daysOff() > 7) {
  approveExtraLeave(employee);
}
```

**After**
```typescript
const WORK_DAYS_PER_WEEK = 7;
if (employee.daysOff() > WORK_DAYS_PER_WEEK) {
  approveExtraLeave(employee);
}
```

## 4. Function does one thing

**Before**
```typescript
function processOrder(order: Order) {
  if (!order.lines.length) throw new Error("empty");
  let total = 0;
  for (const line of order.lines) total += line.price * line.qty;
  order.total = total;
  db.save(order);
  mailer.sendReceipt(order);
}
```

**After**
```typescript
function processOrder(order: Order) {
  validateNonEmpty(order);
  order.total = computeTotal(order.lines);
  persistOrder(order);
  notifyCustomer(order);
}
```

## 5. No side effects in a check

**Before**
```typescript
function checkPassword(user: User, password: string): boolean {
  const ok = hasher.matches(password, user.passwordHash);
  if (ok) session.start(user); // surprise side effect
  return ok;
}
```

**After**
```typescript
function passwordMatches(user: User, password: string): boolean {
  return hasher.matches(password, user.passwordHash);
}

function login(user: User, password: string): void {
  if (!passwordMatches(user, password)) {
    throw new AuthError("bad credentials");
  }
  session.start(user);
}
```

## 6. DRY — repeated error handling

**Before**
```typescript
try {
  await fetchUser(id);
} catch (e) {
  log.error("fetchUser", e);
  throw new AppError("user fetch failed", { cause: e });
}
try {
  await fetchOrders(id);
} catch (e) {
  log.error("fetchOrders", e);
  throw new AppError("orders fetch failed", { cause: e });
}
```

**After**
```typescript
async function withLoggedErrors<T>(op: string, fn: () => Promise<T>): Promise<T> {
  try {
    return await fn();
  } catch (e) {
    log.error(op, e);
    throw new AppError(`${op} failed`, { cause: e });
  }
}
await withLoggedErrors("fetchUser", () => fetchUser(id));
await withLoggedErrors("fetchOrders", () => fetchOrders(id));
```

## 7. Exceptions over return codes

**Before**
```typescript
function deleteFile(path: string): number {
  if (!exists(path)) return -1;
  if (!isWritable(path)) return -2;
  doDelete(path);
  return 0;
}
```

**After**
```typescript
function deleteFile(path: string): void {
  if (!exists(path)) throw new NotFoundError(path);
  if (!isWritable(path)) throw new AccessDeniedError(path);
  doDelete(path);
}
```

## 8. Don’t return null

`db.get` already yields `Employee | undefined`. Widening to `| null` forces callers to handle **two** “missing” shapes for no gain.

**Before**
```typescript
function findEmployee(id: string): Employee | null {
  const row = db.get(id); // Employee | undefined
  return row ?? null; // unnecessary: widens to null and duplicates absence semantics
}
```

**After**
```typescript
function findEmployee(id: string): Employee | undefined {
  return db.get(id); // one clear “not found” representation
}
```

## 9. Single concept / one logical outcome per test

**Before**
```typescript
import { describe, it, expect } from "vitest";

it("userStuff", () => {
  const user = service.register("a@b.com", "secret");
  expect(user.id).not.toBeNull();
  expect(service.login("a@b.com", "secret")).toBe(true);
  expect(service.resetPassword(user.id)).toBe(true);
});
```

**After**
```typescript
import { describe, it, expect } from "vitest";

it("register assigns id", () => {
  const user = service.register("a@b.com", "secret");
  expect(user.id).not.toBeNull();
});

it("login succeeds after register", () => {
  service.register("a@b.com", "secret");
  expect(service.login("a@b.com", "secret")).toBe(true);
});
```

## 10. Polymorphism over switch on type

**Before**
```typescript
function area(shape: Shape) {
  switch (shape.kind) {
    case "circle": return Math.PI * shape.r ** 2;
    case "rect": return shape.w * shape.h;
    default: throw new Error("unknown");
  }
}
```

**After**
```typescript
interface Shape { area(): number; }
class Circle implements Shape {
  constructor(readonly r: number) {}
  area() { return Math.PI * this.r ** 2; }
}
class Rect implements Shape {
  constructor(readonly w: number, readonly h: number) {}
  area() { return this.w * this.h; }
}
```

## 11. Encapsulate conditional

**Before**
```typescript
if (
  employee.isEligibleForFullBenefits() &&
  employee.age > 55 &&
  employee.yearsOfService > 10
) {
  subsidize(employee);
}
```

**After**
```typescript
// isEligibleForSeniorSubsidy encapsulates the compound rule
if (employee.isEligibleForSeniorSubsidy()) {
  subsidize(employee);
}
```

## 12. Magic number → named constant

**Before**
```typescript
function estimateDeliveryDays(distanceKm: number): number {
  return 3 + Math.floor(distanceKm / 500);
}
```

**After**
```typescript
const BASE_SHIPPING_DAYS = 3;
const KM_PER_DAY_CHUNK = 500;

function estimateDeliveryDays(distanceKm: number): number {
  return BASE_SHIPPING_DAYS + Math.floor(distanceKm / KM_PER_DAY_CHUNK);
}
```
