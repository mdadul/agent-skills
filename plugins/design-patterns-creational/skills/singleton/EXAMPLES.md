# Singleton Examples

## 1) Database Connection (Canonical Example)

### Context
The app must share one database connection across all services. Creating multiple connections wastes resources and breaks transaction coordination.

### Design
- Private static `instance` field.
- Private constructor performs the real connection setup.
- `getInstance()` uses double-checked locking (Java/C# with `volatile`).

### Pseudocode
```
class Database
    private static volatile instance: Database

    private constructor()
        // open connection

    public static getInstance(): Database
        if instance == null
            lock(Database.class)
                if instance == null
                    instance = new Database()
        return instance

    public query(sql): ResultSet
        // execute query
```

### Outcome
All services call `Database.getInstance()` and share the same connection. The lock is only contended during the very first call; subsequent calls are lock-free.

---

## 2) Eager Initialization (Config / Logger)

### Context
Application configuration is loaded once at startup from environment variables. Cost is negligible; simplicity is preferred over lazy loading.

### Design
```typescript
class AppConfig {
    private static readonly instance = new AppConfig();

    private constructor() {
        // load env vars
    }

    static getInstance(): AppConfig {
        return AppConfig.instance;
    }
}
```

### Note
Static field initializers run once when the class is first loaded. No locking needed. Prefer this when startup cost is low.

---

## 3) Go — `sync.Once` (Idiomatic)

### Context
Go has no private constructors or static class members. The idiomatic singleton uses a package-level variable and `sync.Once`.

```go
var (
    instance *db
    once     sync.Once
)

func GetDB() *db {
    once.Do(func() {
        instance = &db{} // expensive init
    })
    return instance
}
```

### Note
`sync.Once` guarantees exactly-once execution even under concurrent first calls. Prefer this over hand-rolled double-checked locking in Go.

---

## 4) Kotlin — `object` Declaration

### Context
Kotlin's `object` keyword is a language-level singleton — thread-safe, lazy on first access, no boilerplate.

```kotlin
object AppLogger {
    fun log(message: String) { println(message) }
}

// Usage
AppLogger.log("started")
```

### Note
When using Kotlin, always prefer `object` over a Java-style hand-rolled Singleton.

---

## 5) Testability — Interface + DI Alternative

### Context
A service depends on `Database.getInstance()`. Unit tests can't inject a fake because the static method is not overridable.

### Problem
```typescript
class OrderService {
    save(order: Order) {
        Database.getInstance().query("INSERT ...");
    }
}
```
Tests hit the real database; state leaks between test cases.

### Fix: extract interface + inject
```typescript
interface IDatabase {
    query(sql: string): unknown;
}

class OrderService {
    constructor(private db: IDatabase) {}
    save(order: Order) { this.db.query("INSERT ..."); }
}

// Production wiring (in DI container or main)
const service = new OrderService(Database.getInstance());

// Test wiring
const service = new OrderService(new FakeDatabase());
```

### Note
The singleton instance itself is still managed by `Database`; `OrderService` just receives it via constructor injection. Tests pass a fake without touching the singleton at all.

---

## 6) Quick Evaluation Prompts
- "Implement a thread-safe Singleton for a config loader in Java."
- "Is Singleton appropriate here, or should I use DI?"
- "Show me the initialization-on-demand holder idiom in Java."
- "How do I mock a Singleton in tests without breaking the pattern?"
- "What's the idiomatic Singleton in Go / Kotlin / Python?"
