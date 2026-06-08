# .NET Junior → Entry Mid-Level — 100 Classic Interview Questions

> **Target level:** Junior → Entry Mid-level .NET Software Engineer
> **Topics:** C# Fundamentals, OOP, EF Core, ASP.NET Core, Async/Concurrency, Memory, Collections, Design Patterns, Architecture, Testing, SQL, Security, Git/DevOps

---

## SECTION 1 — C# Fundamentals (Q1–Q15)

---

### Q1. What is the difference between `value types` and `reference types` in C#?

**Strong Answer**
Value types (e.g. `int`, `bool`, `struct`, `enum`) store their data directly on the stack. When you assign one to another variable, a copy is made. Reference types (e.g. `class`, `string`, `array`) store a reference (pointer) on the stack pointing to data on the heap. Assigning copies the reference, not the data — both variables point to the same object.

```csharp
// Value type — copy
int a = 5;
int b = a;
b = 10;
Console.WriteLine(a); // 5 — unchanged

// Reference type — shared reference
var list1 = new List<int> { 1, 2, 3 };
var list2 = list1;
list2.Add(4);
Console.WriteLine(list1.Count); // 4 — affected
```

**Keywords:** stack, heap, copy semantics, reference semantics, boxing, unboxing

---

### Q2. What is boxing and unboxing, and why is it a performance concern?

**Strong Answer**
Boxing is converting a value type to `object` (heap allocation). Unboxing is extracting the value type back from `object`. Each boxing operation allocates a new heap object — in tight loops this causes GC pressure.

```csharp
int x = 42;
object boxed = x;         // boxing — heap allocation
int unboxed = (int)boxed; // unboxing — cast required
```

Common hidden boxing: using value types in non-generic collections like `ArrayList`, or passing structs to methods accepting `object`.

**Keywords:** heap allocation, GC pressure, non-generic collections, implicit boxing

---

### Q3. What is the difference between `string` and `StringBuilder`?

**Strong Answer**
`string` is immutable — every concatenation creates a new string object on the heap. In a loop this means N allocations for N concatenations. `StringBuilder` uses a mutable internal buffer, appending in place — one allocation for the builder, one for the final string.

```csharp
// BAD in loops — N heap allocations
string result = "";
for (int i = 0; i < 10000; i++)
    result += i.ToString();

// GOOD — single buffer
var sb = new StringBuilder();
for (int i = 0; i < 10000; i++)
    sb.Append(i);
string result = sb.ToString();
```

**Keywords:** immutability, heap allocation, internal buffer, GC pressure, string interning

---

### Q4. What is the difference between `==` and `.Equals()` in C#?

**Strong Answer**
For value types, both compare values. For reference types, `==` by default compares references (same object in memory). `.Equals()` can be overridden to compare values — `string` overrides both to compare content.

```csharp
string a = new string("hello".ToCharArray());
string b = new string("hello".ToCharArray());

Console.WriteLine(a == b);                       // true — string overrides ==
Console.WriteLine(a.Equals(b));                  // true — value comparison
Console.WriteLine(object.ReferenceEquals(a, b)); // false — different objects
```

**Keywords:** reference equality, value equality, operator overloading, ReferenceEquals, string override

---

### Q5. What is the null coalescing (`??`) and null conditional (`?.`) operator?

**Strong Answer**
`??` returns the left operand if not null, otherwise the right. `?.` short-circuits to null if the left side is null instead of throwing `NullReferenceException`.

```csharp
string name = null;

string display = name ?? "Anonymous"; // "Anonymous"
name ??= "Default";                   // assigns only if null

int? length = name?.Length;           // null, not exception

string city = user?.Address?.City ?? "Unknown"; // chaining
```

**Keywords:** null safety, NullReferenceException, short-circuit, null coalescing assignment

---

### Q6. What is the difference between `readonly` and `const`?

**Strong Answer**
`const` is a compile-time constant — value is inlined at every usage site during compilation, must be a primitive or string, implicitly static. `readonly` is a runtime constant — assigned once at declaration or in the constructor, can hold any type including complex objects.

```csharp
public class Config
{
    public const int MaxRetries = 3;       // compile-time, inlined
    public readonly DateTime StartedAt;    // runtime, set in constructor

    public Config()
    {
        StartedAt = DateTime.UtcNow;       // allowed
        // MaxRetries = 5;                 // compile error
    }
}
```

**Keywords:** compile-time constant, runtime constant, inlining, immutability, constructor assignment

---

### Q7. Explain `static` classes and methods. When would you use them?

**Strong Answer**
A `static` class cannot be instantiated — all members must be static. A `static` method belongs to the type, not an instance. Use for stateless utility logic — helpers, extension methods, factory methods — where no instance state is needed.

```csharp
public static class StringHelper
{
    public static string ToCamelCase(string input) { /* ... */ }
}

public static class OrderExtensions
{
    public static bool IsExpired(this Order order) =>
        order.ExpiresAt < DateTime.UtcNow;
}
```

Avoid overusing static for logic that needs to be mocked or tested — static methods are hard to substitute in unit tests.

**Keywords:** stateless, utility, extension methods, no instantiation, testability concern

---

### Q8. What are `abstract` classes vs `interfaces`? When do you use each?

**Strong Answer**
An `abstract` class can have implementation, fields, constructors, and access modifiers — used when classes share code and represent an "is-a" hierarchy. An `interface` defines a contract — used for capability contracts across unrelated types. A class can implement multiple interfaces but only inherit one abstract class.

```csharp
public abstract class Animal
{
    public string Name { get; set; }
    public abstract void Speak();
    public void Breathe() { /* shared implementation */ }
}

public interface IExportable
{
    Task ExportAsync(Stream output);
}
```

**Keywords:** is-a relationship, capability contract, multiple interfaces, single inheritance, default interface methods

---

### Q9. What is the difference between `override`, `new`, and `virtual`?

**Strong Answer**
`virtual` marks a base method as overridable. `override` replaces it in a derived class — polymorphism works correctly. `new` hides the base method in the derived class but does NOT participate in polymorphism — calling through a base reference still calls the base method.

```csharp
class Animal  { public virtual void Speak() => Console.WriteLine("..."); }
class Dog : Animal { public override void Speak() => Console.WriteLine("Woof"); }
class Cat : Animal { public new void Speak() => Console.WriteLine("Meow"); }

Animal a = new Dog(); a.Speak(); // "Woof" — override is polymorphic
Animal b = new Cat(); b.Speak(); // "..."  — new is NOT polymorphic
```

**Keywords:** polymorphism, method hiding, virtual dispatch, vtable

---

### Q10. What is a `delegate`, `Func`, and `Action`?

**Strong Answer**
A `delegate` is a type-safe function pointer. `Func<T, TResult>` is a built-in delegate that takes parameters and returns a value. `Action<T>` is a built-in delegate that takes parameters and returns void.

```csharp
Func<int, int, int> multiply = (a, b) => a * b;
Action<string> log = message => Console.WriteLine(message);

// Common in LINQ
var evens = numbers.Where(n => n % 2 == 0); // Func<int, bool>
```

**Keywords:** type-safe function pointer, lambda, higher-order functions, LINQ predicates

---

### Q11. What is the difference between `IEnumerable`, `ICollection`, and `IList`?

**Strong Answer**
These form a hierarchy — each adds capabilities:

- `IEnumerable<T>` — forward-only iteration. No count, no add, no index access.
- `ICollection<T>` — adds `Count`, `Add`, `Remove`, `Contains`.
- `IList<T>` — adds index access `[i]`, `Insert`, `RemoveAt`.

Program to the narrowest interface your code needs — return `IEnumerable` if callers only need to iterate.

**Keywords:** interface segregation, forward-only, index access, program to interface

---

### Q12. What is `yield return` and how does it work?

**Strong Answer**
`yield return` creates a lazy iterator — the method pauses at each `yield` and resumes on the next iteration. No list is built; values are produced one at a time. Memory-efficient for large sequences.

```csharp
IEnumerable<int> GetEvenNumbers(int max)
{
    for (int i = 0; i <= max; i++)
        if (i % 2 == 0)
            yield return i;
}

foreach (var n in GetEvenNumbers(1_000_000))
{
    if (n > 10) break; // stops at 12, never computes 14–1M
}
```

**Keywords:** lazy evaluation, iterator, state machine, deferred execution, memory efficiency

---

### Q13. What is the difference between `throw` and `throw ex`?

**Strong Answer**
`throw` re-throws the current exception preserving the original stack trace. `throw ex` re-throws but resets the stack trace to the current line — you lose the original call stack, making debugging much harder.

```csharp
try { RiskyOperation(); }
catch (Exception ex)
{
    Log(ex);
    throw;      // GOOD — preserves full stack trace
    // throw ex; // BAD — stack trace starts here, origin lost
}
```

**Keywords:** stack trace, exception propagation, inner exception, debugging

---

### Q14. What is `IDisposable` and the `using` statement?

**Strong Answer**
`IDisposable` defines a `Dispose()` method for deterministic cleanup of unmanaged resources (DB connections, file handles, streams). The `using` statement guarantees `Dispose()` is called even if an exception is thrown — it compiles to a `try/finally` block.

```csharp
using (var conn = new SqlConnection(connectionString))
{
    conn.Open();
} // Dispose() called — connection returned to pool

// C#8 using declaration
using var conn = new SqlConnection(connectionString);
// Dispose() called at end of enclosing scope
```

**Keywords:** unmanaged resources, deterministic cleanup, try/finally, connection pool, finalizer

---

### Q15. What are generics and why are they better than using `object`?

**Strong Answer**
Generics allow type-safe, reusable code without boxing. Using `object` loses compile-time type safety and causes boxing for value types.

```csharp
// Without generics — loses type safety + boxing
ArrayList list = new ArrayList();
list.Add(42);           // boxing
int x = (int)list[0];  // unboxing + cast, runtime error risk

// With generics — type-safe, no boxing
List<int> list = new List<int>();
list.Add(42);  // no boxing
int x = list[0]; // no cast, compile-time safe
```

**Keywords:** type safety, compile-time checking, boxing avoidance, reusability, type parameter constraints

---

## SECTION 2 — OOP & Design (Q16–Q25)

---

### Q16. What are the four pillars of OOP?

**Strong Answer**
- **Encapsulation** — hide internal state, expose via public API. Protects invariants.
- **Abstraction** — expose only what's needed, hide complexity.
- **Inheritance** — derive new types from existing ones, reuse and extend behavior.
- **Polymorphism** — same interface, different implementations. Runtime dispatch via virtual/override.

**Keywords:** encapsulation, abstraction, inheritance, polymorphism, invariants

---

### Q17. What is the SOLID principle? Explain each briefly.

**Strong Answer**
- **S** — Single Responsibility: one reason to change.
- **O** — Open/Closed: open for extension, closed for modification.
- **L** — Liskov Substitution: derived classes substitutable for base.
- **I** — Interface Segregation: many small interfaces over one large one.
- **D** — Dependency Inversion: depend on abstractions, not concretions.

```csharp
// D — Dependency Inversion
public class OrderService
{
    private readonly IOrderRepository _repo; // abstraction
    public OrderService(IOrderRepository repo) => _repo = repo;
}
```

**Keywords:** single responsibility, open/closed, Liskov, interface segregation, dependency inversion

---

### Q18. What is Dependency Injection and why is it important?

**Strong Answer**
DI provides dependencies to a class rather than letting the class create them. ASP.NET Core's built-in IoC container manages lifetimes and injects via constructors. Enables loose coupling, testability, and replaceability.

```csharp
builder.Services.AddScoped<IOrderService, OrderService>();

public class OrderController : ControllerBase
{
    private readonly IOrderService _orderService;
    public OrderController(IOrderService orderService)
        => _orderService = orderService;
}
```

**Keywords:** IoC container, loose coupling, constructor injection, lifetime, testability

---

### Q19. What is the difference between Singleton, Scoped, and Transient lifetimes in ASP.NET Core DI?

**Strong Answer**
- **Singleton** — one instance for the entire app lifetime.
- **Scoped** — one instance per HTTP request. Correct for DbContext.
- **Transient** — new instance every time it's requested.

```csharp
builder.Services.AddSingleton<ICache, MemoryCache>();
builder.Services.AddScoped<AppDbContext>();
builder.Services.AddTransient<IEmailValidator>();
```

Common pitfall: injecting a Scoped service into a Singleton — the Scoped instance gets captured and lives too long (captive dependency).

**Keywords:** lifetime management, captive dependency, DbContext per request, IoC

---

### Q20. What is the Repository pattern and why use it?

**Strong Answer**
Repository abstracts data access behind an interface — business logic talks to `IOrderRepository`, not `DbContext` directly. Decouples domain from ORM, enables testing with mocks, centralizes query logic.

```csharp
public interface IOrderRepository
{
    Task<Order> GetByIdAsync(int id);
    Task AddAsync(Order order);
}

public class EfOrderRepository : IOrderRepository
{
    private readonly AppDbContext _ctx;
    public async Task<Order> GetByIdAsync(int id) =>
        await _ctx.Orders.AsNoTracking().FirstOrDefaultAsync(o => o.Id == id);
}
```

**Keywords:** abstraction over data access, testability, separation of concerns, Unit of Work

---

### Q21. What is the Unit of Work pattern?

**Strong Answer**
Unit of Work groups multiple repository operations into one transaction — either all succeed or all fail. In EF Core, `DbContext` itself is a Unit of Work — `SaveChanges()` commits all tracked changes atomically.

```csharp
await _uow.Orders.AddAsync(order);
await _uow.Products.UpdateStockAsync(productId, -quantity);
await _uow.SaveChangesAsync(); // both committed atomically
```

**Keywords:** atomic transaction, DbContext as UoW, consistency, rollback

---

### Q22. What is the difference between composition and inheritance?

**Strong Answer**
Inheritance is "is-a" — `Dog` is an `Animal`. Composition is "has-a" — `Logger` has a `Writer`. Composition is generally preferred — more flexible, avoids the fragile base class problem, easier to test.

```csharp
// Composition — flexible
class Logger
{
    private readonly IWriter _writer;
    public Logger(IWriter writer) => _writer = writer;
}
```

**Keywords:** is-a vs has-a, fragile base class, favor composition over inheritance, flexibility

---

### Q23. What is the Factory pattern?

**Strong Answer**
Factory encapsulates object creation — callers get an instance without knowing the concrete type. Useful when creation logic is complex or the type depends on runtime conditions.

```csharp
public class NotificationFactory
{
    public INotificationSender Create(string channel) => channel switch
    {
        "email" => new EmailSender(),
        "sms"   => new SmsSender(),
        _ => throw new ArgumentException($"Unknown: {channel}")
    };
}
```

**Keywords:** encapsulate creation, decouple from concrete type, open/closed

---

### Q24. What is the Strategy pattern?

**Strong Answer**
Strategy defines a family of algorithms, encapsulates each, and makes them interchangeable — the context delegates behavior to a strategy interface.

```csharp
public interface IDiscountStrategy { decimal Apply(decimal price); }

public class OrderProcessor
{
    private readonly IDiscountStrategy _discount;
    public OrderProcessor(IDiscountStrategy discount) => _discount = discount;
    public decimal Process(decimal price) => _discount.Apply(price);
}
```

**Keywords:** interchangeable algorithms, open/closed, behavior delegation, DI-friendly

---

### Q25. What is the Observer pattern?

**Strong Answer**
Observer defines a one-to-many dependency — when one object changes state, all dependents are notified. In .NET this maps to `event`/`delegate` or domain events.

```csharp
public class OrderService
{
    public event EventHandler<OrderPlacedEventArgs> OrderPlaced;

    public async Task PlaceOrderAsync(Order order)
    {
        await SaveOrderAsync(order);
        OrderPlaced?.Invoke(this, new OrderPlacedEventArgs(order));
    }
}
```

**Keywords:** event-driven, publish/subscribe, loose coupling, EventHandler, domain events

---

## SECTION 3 — Async & Concurrency (Q26–Q38)

---

### Q26. What is the difference between synchronous and asynchronous code?

**Strong Answer**
Synchronous code blocks the thread until the operation completes. Asynchronous code releases the thread while waiting for I/O — improving throughput under load in ASP.NET Core.

```csharp
// Sync — blocks thread during DB call
public Order GetOrder(int id) => _ctx.Orders.Find(id);

// Async — releases thread while DB query runs
public async Task<Order> GetOrderAsync(int id) =>
    await _ctx.Orders.FindAsync(id);
```

**Keywords:** thread blocking, I/O-bound, thread pool, throughput, async/await, Task

---

### Q27. What does `async`/`await` actually do under the hood?

**Strong Answer**
The compiler transforms an `async` method into a state machine. At each `await`, if the awaited task is not complete, the method returns control to the caller and registers a continuation. When the task completes, the continuation resumes the method — no thread is blocked in between.

**Keywords:** state machine, continuation, task scheduler, SynchronizationContext, compiler transformation

---

### Q28. What is the difference between `Task.Run()` and `await`?

**Strong Answer**
`await` is for I/O-bound async operations — releases the current thread, no new thread spawned. `Task.Run()` offloads CPU-bound work to a thread pool thread — use for heavy computation that would block the calling thread.

```csharp
var data = await httpClient.GetStringAsync(url);          // I/O — no new thread
var result = await Task.Run(() => HeavyComputation(data)); // CPU — uses thread pool
```

**Keywords:** I/O-bound vs CPU-bound, thread pool, offloading

---

### Q29. What is a deadlock in async code and how do you avoid it?

**Strong Answer**
A deadlock occurs when `.Result` or `.Wait()` is called on a Task in a context with a `SynchronizationContext`. The calling thread blocks waiting for the task, but the task's continuation needs the same thread to resume — circular wait.

```csharp
// BAD — can deadlock
var result = GetDataAsync().Result;

// GOOD — always await
var result = await GetDataAsync();

// Library code — prevent context capture
var result = await GetDataAsync().ConfigureAwait(false);
```

**Keywords:** SynchronizationContext, deadlock, ConfigureAwait(false), .Result anti-pattern

---

### Q30. What is `CancellationToken` and when should you use it?

**Strong Answer**
`CancellationToken` allows cooperative cancellation — the caller signals, the callee checks and throws `OperationCanceledException`. Always accept and pass it in long-running async methods.

```csharp
public async Task<List<Product>> SearchAsync(string query, CancellationToken ct = default)
{
    return await _ctx.Products
        .Where(p => p.Name.Contains(query))
        .ToListAsync(ct);
}
```

**Keywords:** cooperative cancellation, OperationCanceledException, timeout, HttpContext.RequestAborted

---

### Q31. What is `Task.WhenAll` vs `Task.WhenAny`?

**Strong Answer**
`Task.WhenAll` completes when ALL tasks finish. `Task.WhenAny` completes when the FIRST task finishes — useful for racing or timeout patterns.

```csharp
// WhenAll — need all results
await Task.WhenAll(task1, task2, task3);

// WhenAny — timeout pattern
var winner = await Task.WhenAny(dataTask, Task.Delay(5000));
if (winner != dataTask) throw new TimeoutException();
```

**Keywords:** parallel execution, racing, timeout, aggregated exceptions

---

### Q32. What is `SemaphoreSlim` and when do you use it?

**Strong Answer**
`SemaphoreSlim` limits how many tasks can execute a section concurrently. Use it to bound concurrent operations when `Task.WhenAll` would otherwise fire too many tasks at once.

```csharp
var semaphore = new SemaphoreSlim(10);

var tasks = items.Select(async item =>
{
    await semaphore.WaitAsync();
    try { await ProcessAsync(item); }
    finally { semaphore.Release(); }
});

await Task.WhenAll(tasks);
```

**Keywords:** bounded concurrency, connection pool protection, WaitAsync, throttling

---

### Q33. What is `Parallel.ForEachAsync` and how does it differ from `Task.WhenAll`?

**Strong Answer**
`Parallel.ForEachAsync` (.NET 6+) has built-in `MaxDegreeOfParallelism` and `CancellationToken` support. `Task.WhenAll` with `SemaphoreSlim` achieves the same but with more boilerplate.

```csharp
await Parallel.ForEachAsync(items,
    new ParallelOptions { MaxDegreeOfParallelism = 10 },
    async (item, ct) => await ProcessAsync(item));
```

**Keywords:** MaxDegreeOfParallelism, .NET 6+, bounded concurrency, built-in cancellation

---

### Q34. What is `IAsyncEnumerable<T>` and when do you use it?

**Strong Answer**
`IAsyncEnumerable<T>` allows async iteration with `await foreach`. Use when producing or consuming a sequence of items asynchronously one at a time — streaming DB rows, paginated API results.

```csharp
public async IAsyncEnumerable<Product> StreamProductsAsync(
    [EnumeratorCancellation] CancellationToken ct = default)
{
    await foreach (var p in _ctx.Products.AsAsyncEnumerable().WithCancellation(ct))
        yield return p;
}
```

**Keywords:** async streaming, await foreach, yield return, EnumeratorCancellation, lazy async sequence

---

### Q35. What is a `lock` statement and when do you use it?

**Strong Answer**
`lock` provides mutual exclusion — only one thread executes the locked block at a time. For async code, use `SemaphoreSlim(1,1)` instead — you cannot `await` inside a `lock`.

```csharp
private readonly object _lock = new object();
private int _counter = 0;

public void Increment()
{
    lock (_lock) { _counter++; }
}
```

**Keywords:** mutual exclusion, thread safety, monitor, SemaphoreSlim for async, deadlock risk

---

### Q36. What is the difference between optimistic and pessimistic concurrency?

**Strong Answer**
Pessimistic — lock the row on read, no one else can modify until you're done. Optimistic — no lock; on update check if the row changed (via `RowVersion`), reject if it did.

```csharp
public class Product
{
    public int Id { get; set; }
    public decimal Price { get; set; }
    [Timestamp]
    public byte[] RowVersion { get; set; }
}

try { await _ctx.SaveChangesAsync(); }
catch (DbUpdateConcurrencyException) { /* handle conflict */ }
```

**Keywords:** RowVersion, Timestamp, DbUpdateConcurrencyException, conflict detection, throughput vs safety

---

### Q37. What is `ConfigureAwait(false)` and when should you use it?

**Strong Answer**
`ConfigureAwait(false)` tells the runtime not to resume on the original `SynchronizationContext` — the continuation runs on any thread pool thread. Prevents deadlocks in library code, slightly faster.

```csharp
// Library code
public async Task<string> FetchAsync(string url)
{
    var response = await _http.GetAsync(url).ConfigureAwait(false);
    return await response.Content.ReadAsStringAsync().ConfigureAwait(false);
}
```

**Keywords:** SynchronizationContext, thread pool, deadlock prevention, library code best practice

---

### Q38. What is `Channel<T>` and when would you use it?

**Strong Answer**
`Channel<T>` is an in-process async producer-consumer pipeline with backpressure support. Unlike a queue, it's async-native — no polling, no blocking.

```csharp
var channel = Channel.CreateBounded<WorkItem>(100);

// Producer
await channel.Writer.WriteAsync(new WorkItem());

// Consumer
await foreach (var item in channel.Reader.ReadAllAsync())
    await ProcessAsync(item);
```

Use for in-process pipelines. Use RabbitMQ/Azure Service Bus for distributed messaging.

**Keywords:** producer-consumer, backpressure, bounded channel, async-native, in-process

---

## SECTION 4 — EF Core & Database (Q39–Q52)

---

### Q39. What is EF Core and what does it do?

**Strong Answer**
EF Core is an ORM — maps C# classes to database tables and translates LINQ queries into SQL. It manages DbContext (unit of work), change tracking, migrations, and entity relationships.

**Keywords:** ORM, LINQ to SQL translation, DbContext, entity mapping, migrations

---

### Q40. What is the N+1 query problem in EF Core?

**Strong Answer**
N+1 occurs when you load N parent entities then issue one query per parent for related children. Fix with `Include()` to load related data in one JOIN.

```csharp
// BAD — 1 + N queries
var orders = await _ctx.Orders.ToListAsync();
foreach (var order in orders) { var items = order.Items; }

// GOOD — 1 query with JOIN
var orders = await _ctx.Orders.Include(o => o.Items).ToListAsync();
```

**Keywords:** eager loading, lazy loading, Include, N+1, SELECT N+1, JOIN

---

### Q41. What is the difference between eager, lazy, and explicit loading in EF Core?

**Strong Answer**
- **Eager** — `Include()` — same query, JOIN.
- **Lazy** — loads automatically on access via proxy — N+1 risk.
- **Explicit** — `Entry().Collection().LoadAsync()` — manual, on demand.

**Keywords:** Include, ThenInclude, virtual navigation, proxy, explicit load, N+1 risk

---

### Q42. What are EF Core migrations and how do they work?

**Strong Answer**
Migrations track model changes as versioned C# files — `Up()` applies, `Down()` rolls back. EF diffs the current model against the last migration snapshot.

```bash
dotnet ef migrations add AddProductCategory
dotnet ef database update
```

**Keywords:** model snapshot, Up/Down, migration history table, idempotent, dotnet ef CLI

---

### Q43. What is the difference between `FirstOrDefault`, `SingleOrDefault`, and `Find`?

**Strong Answer**
- `FirstOrDefault` — first match or null. Multiple matches OK.
- `SingleOrDefault` — exactly one match or null. Throws if multiple.
- `Find` — PK lookup, checks change tracker cache first — fastest for PK.

**Keywords:** single vs first, InvalidOperationException, change tracker cache, PK lookup

---

### Q44. What is the difference between `SaveChanges` and `ExecuteUpdateAsync`?

**Strong Answer**
`SaveChanges` loads entities into memory, tracks changes, issues per-entity SQL. `ExecuteUpdateAsync` (EF Core 7+) runs a single SQL UPDATE without loading entities — far more efficient for bulk operations.

```csharp
await _ctx.Products
    .Where(p => p.CategoryId == 5)
    .ExecuteUpdateAsync(p =>
        p.SetProperty(x => x.Price, x => x.Price * 0.9m));
```

**Keywords:** bulk update, change tracker bypass, EF Core 7+, single SQL, SetProperty

---

### Q45. How do you handle database transactions in EF Core?

**Strong Answer**
`SaveChanges` wraps changes in one transaction by default. For multiple `SaveChanges` calls, use explicit transactions.

```csharp
await using var transaction = await _ctx.Database.BeginTransactionAsync();
try
{
    await _ctx.SaveChangesAsync();
    await _ctx.SaveChangesAsync();
    await transaction.CommitAsync();
}
catch { await transaction.RollbackAsync(); throw; }
```

**Keywords:** atomicity, BeginTransaction, CommitAsync, RollbackAsync

---

### Q46. What is a database index and why does it matter?

**Strong Answer**
An index is a B-tree structure the DB maintains alongside the table — enables O(log n) seeks instead of O(n) full table scans. PKs are auto-indexed. Foreign keys and frequently filtered columns need manual indexes.

```csharp
modelBuilder.Entity<Order>()
    .HasIndex(o => new { o.CustomerId, o.Status });
```

**Keywords:** B-tree, full table scan, seek, composite index, covering index

---

### Q47. What is SQL injection and how does EF Core protect against it?

**Strong Answer**
SQL injection embeds user input directly into SQL — attacker changes query logic. EF Core uses parameterized queries by default — values passed as SQL parameters, never interpolated.

```csharp
// BAD — injection risk
_ctx.Database.ExecuteSqlRaw($"SELECT * FROM Users WHERE Name = '{userInput}'");

// GOOD — EF LINQ (parameterized automatically)
_ctx.Users.Where(u => u.Name == userInput);
```

**Keywords:** parameterized query, SQL parameter, LINQ is safe by default

---

### Q48. What is the difference between `Include` and `ThenInclude`?

**Strong Answer**
`Include` loads a direct navigation property. `ThenInclude` chains off the included property to load a deeper level.

```csharp
var orders = await _ctx.Orders
    .Include(o => o.Items)
        .ThenInclude(i => i.Product)
    .ToListAsync();
```

**Keywords:** navigation property, object graph, eager loading, multi-level include

---

### Q49. What is the difference between `DbContext` scoping in a web app vs background service?

**Strong Answer**
In ASP.NET Core, `DbContext` is Scoped — one per request. In a `BackgroundService` there is no request scope — create scopes manually with `IServiceScopeFactory`.

```csharp
public class ReportWorker : BackgroundService
{
    private readonly IServiceScopeFactory _scopeFactory;

    protected override async Task ExecuteAsync(CancellationToken ct)
    {
        using var scope = _scopeFactory.CreateScope();
        var ctx = scope.ServiceProvider.GetRequiredService<AppDbContext>();
    }
}
```

**Keywords:** IServiceScopeFactory, captive dependency, Scoped lifetime, manual scope

---

### Q50. What is connection pooling and how does EF Core use it?

**Strong Answer**
Connection pooling reuses physical DB connections — opening connections is expensive. `Dispose()` on a connection returns it to the pool, not closes it. Scoped `DbContext` ensures prompt return at request end.

**Keywords:** ADO.NET pool, connection reuse, Dispose returns to pool, pool exhaustion

---

### Q51. What is a covering index?

**Strong Answer**
A covering index includes all columns needed to satisfy a query — the DB answers entirely from the index without touching the main table (no key lookup).

```csharp
modelBuilder.Entity<Product>()
    .HasIndex(p => p.CategoryId)
    .IncludeProperties(p => new { p.Name, p.Price });
```

**Keywords:** key lookup elimination, INCLUDE columns, I/O reduction, index-only scan

---

### Q52. What is the difference between `FromSqlRaw` and `FromSqlInterpolated`?

**Strong Answer**
`FromSqlRaw` uses positional parameters. `FromSqlInterpolated` uses C# string interpolation but EF Core auto-converts to SQL parameters — both safe. Never use raw string concatenation.

```csharp
// Safe
_ctx.Products.FromSqlInterpolated($"SELECT * FROM Products WHERE CategoryId = {categoryId}");

// Unsafe — injection risk
_ctx.Products.FromSqlRaw("SELECT * FROM Products WHERE CategoryId = " + categoryId);
```

**Keywords:** parameterized, FormattableString, injection-safe, raw SQL in EF

---

## SECTION 5 — ASP.NET Core (Q53–Q65)

---

### Q53. What is middleware in ASP.NET Core and how does it work?

**Strong Answer**
Middleware is a pipeline of components that process HTTP requests and responses. Each component can inspect, modify, short-circuit, or pass to the next. Order of registration matters.

```csharp
app.UseExceptionHandler();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();

app.Use(async (context, next) => {
    // before
    await next(context);
    // after
});
```

**Keywords:** pipeline, request/response, short-circuit, order matters, Use/Run/Map

---

### Q54. What is the difference between `app.Use()`, `app.Run()`, and `app.Map()`?

**Strong Answer**
- `Use()` — calls `next()`, passes to next middleware.
- `Run()` — terminal, does not call `next()`.
- `Map()` — branches pipeline based on path.

**Keywords:** terminal, branching, next delegate, pipeline termination

---

### Q55. What is the difference between authentication and authorization?

**Strong Answer**
Authentication — "who are you?" — verifies identity via JWT, cookie, API key. Authorization — "what can you do?" — checks if the authenticated identity has permission.

```csharp
[Authorize(Roles = "Admin")]
[HttpDelete("{id}")]
public async Task<IActionResult> Delete(int id) { /* ... */ }
```

**Keywords:** JWT, Bearer token, claims, roles, policies, [Authorize], HttpContext.User

---

### Q56. What is JWT and how does it work?

**Strong Answer**
JWT is a self-contained signed token: Header (algorithm), Payload (claims), Signature. Server issues on login; client sends in `Authorization: Bearer` header. Server validates signature — no DB lookup needed. Trade-off: stateless, can't revoke until expiry.

**Keywords:** Header/Payload/Signature, HMAC/RSA signing, stateless, claims, expiry, refresh token

---

### Q57. What is the difference between `[FromBody]`, `[FromQuery]`, and `[FromRoute]`?

**Strong Answer**
```csharp
[HttpGet("{id}")]
public Task<IActionResult> Get([FromRoute] int id) { }

[HttpGet]
public Task<IActionResult> List([FromQuery] int page) { }

[HttpPost]
public Task<IActionResult> Create([FromBody] CreateOrderRequest req) { }
```

**Keywords:** model binding, route parameter, query string, request body, JSON deserialization

---

### Q58. What is the difference between `IActionResult` and `ActionResult<T>`?

**Strong Answer**
`IActionResult` is non-generic — no schema inference. `ActionResult<T>` is generic — enables Swagger to infer response type, allows implicit conversion from `T`.

```csharp
public async Task<ActionResult<OrderDto>> Get(int id)
{
    var order = await _service.GetAsync(id);
    if (order == null) return NotFound();
    return order; // implicit Ok(order)
}
```

**Keywords:** generic return type, OpenAPI schema, implicit conversion, Swagger

---

### Q59. What is model validation in ASP.NET Core?

**Strong Answer**
Data Annotations validate model properties before the action runs. `[ApiController]` automatically returns 400 on invalid models.

```csharp
public class CreateOrderRequest
{
    [Required]
    public int CustomerId { get; set; }

    [Range(0.01, double.MaxValue)]
    public decimal Total { get; set; }
}
```

**Keywords:** Data Annotations, ModelState, [ApiController], automatic 400, FluentValidation

---

### Q60. What is the difference between `AddSingleton`, `AddScoped`, `AddTransient` for a DbContext?

**Strong Answer**
`DbContext` must be Scoped. Singleton DbContext is shared across requests — not thread-safe, causes race conditions. Transient creates new instances per injection — breaks Unit of Work.

**Keywords:** DbContext thread safety, Unit of Work, Scoped, captive dependency

---

### Q61. What is `IHttpClientFactory` and why use it over `new HttpClient()`?

**Strong Answer**
`new HttpClient()` doesn't respect DNS changes and exhausts sockets if not properly disposed. `IHttpClientFactory` manages lifetimes, rotates `HttpMessageHandler`, integrates with Polly.

```csharp
builder.Services.AddHttpClient<IWeatherService, WeatherService>(client =>
    client.BaseAddress = new Uri("https://api.weather.com"));
```

**Keywords:** socket exhaustion, DNS refresh, HttpMessageHandler, Polly, typed clients

---

### Q62. What is `IMemoryCache` and how do you implement cache-aside?

**Strong Answer**
Check cache first, on miss fetch from DB and populate with TTL.

```csharp
if (_cache.TryGetValue($"product:{id}", out Product cached))
    return cached;

var product = await _ctx.Products.FindAsync(id);
_cache.Set($"product:{id}", product, TimeSpan.FromMinutes(10));
return product;
```

**Keywords:** cache-aside, TTL, cache invalidation, stale data, IDistributedCache, Redis

---

### Q63. What is the difference between `200 OK`, `201 Created`, `202 Accepted`, `204 No Content`?

**Strong Answer**
- `200 OK` — succeeded, response body has result.
- `201 Created` — resource created, `Location` header points to it.
- `202 Accepted` — async job accepted, not yet complete.
- `204 No Content` — success, no body (DELETE, PUT).

**Keywords:** HTTP semantics, RESTful, Location header, async job, idempotency

---

### Q64. What is CORS and how do you configure it?

**Strong Answer**
CORS is a browser mechanism blocking requests from different origins. Server must explicitly allow cross-origin requests via headers.

```csharp
builder.Services.AddCors(options =>
    options.AddPolicy("AllowFrontend", policy =>
        policy.WithOrigins("https://myapp.com")
              .AllowAnyHeader().AllowAnyMethod()));

app.UseCors("AllowFrontend");
```

**Keywords:** same-origin policy, preflight, Access-Control-Allow-Origin, OPTIONS, browser enforcement

---

### Q65. What is the difference between REST and gRPC?

**Strong Answer**
REST — HTTP/1.1, JSON, text-based, human readable, good for public APIs. gRPC — HTTP/2, Protocol Buffers (binary), strongly typed contracts, faster, better for internal microservices.

**Keywords:** HTTP/2, Protocol Buffers, binary serialization, strongly typed, streaming, internal vs public API

---

## SECTION 6 — Memory & Performance (Q66–Q72)

---

### Q66. How does the .NET Garbage Collector work?

**Strong Answer**
GC manages heap in three generations. Gen0 — short-lived, collected frequently. Gen1 — medium-lived. Gen2 — long-lived, collected rarely and expensively. Objects surviving a collection are promoted. Objects >= 85KB go to the Large Object Heap (LOH) — collected only with Gen2.

**Keywords:** Gen0/Gen1/Gen2, promotion, LOH, GC pressure, finalization, Dispose, STW pause

---

### Q67. What is `Span<T>` and when would you use it?

**Strong Answer**
`Span<T>` represents a contiguous memory region without allocating. Use for parsing and slicing to avoid substring/array copies.

```csharp
ReadOnlySpan<char> span = "2024-01-15".AsSpan();
int year = int.Parse(span.Slice(0, 4)); // zero allocation
```

**Keywords:** zero allocation, stack-only, slicing, parsing, ref struct, ReadOnlySpan

---

### Q68. What causes memory leaks in .NET and how do you find them?

**Strong Answer**
Common causes: static collections that grow indefinitely, event handlers not unsubscribed, DbContext not disposed, caches without TTL.

```csharp
// Event handler leak
publisher.OnDataReceived += Handler; // must unsubscribe:
publisher.OnDataReceived -= Handler;
```

Tools: dotnet-counters, dotnet-dump, Visual Studio Diagnostics, dotMemory.

**Keywords:** GC roots, event handler leak, static references, IDisposable, dotnet-counters, memory profiler

---

### Q69. What is object pooling and when do you use it?

**Strong Answer**
Object pooling reuses expensive objects instead of creating and GCing them repeatedly. `ArrayPool<T>` for byte arrays in hot paths.

```csharp
byte[] buffer = ArrayPool<byte>.Shared.Rent(4096);
try { /* use */ }
finally { ArrayPool<byte>.Shared.Return(buffer); }
```

**Keywords:** ArrayPool, ObjectPool, GC avoidance, rent/return, hot path optimization

---

### Q70. What is the difference between `struct` and `class` in terms of performance?

**Strong Answer**
Structs — stack allocated (as locals), no GC, copied on assignment. Good for small, short-lived, immutable data. Classes — heap, GC managed. Large or mutable structs cause expensive copies.

```csharp
public readonly struct Point
{
    public float X { get; init; }
    public float Y { get; init; }
}
```

**Keywords:** stack vs heap, copy semantics, GC pressure, readonly struct, boxing risk

---

### Q71. What is a `record` type in C# and when do you use it?

**Strong Answer**
`record` is a reference type with value-based equality, immutability by default, and `with` expression for non-destructive mutation. Perfect for DTOs and value objects.

```csharp
public record OrderDto(int Id, decimal Total, string CustomerName);

var a = new OrderDto(1, 100m, "Lucas");
var b = new OrderDto(1, 100m, "Lucas");
Console.WriteLine(a == b); // true — value equality

var updated = a with { Total = 150m };
```

**Keywords:** value equality, immutability, with expression, deconstruct, DTO, value object

---

### Q72. What is `ArrayPool` and why is it better than `new byte[]` in hot paths?

**Strong Answer**
`new byte[]` in a hot path generates frequent short-lived heap objects — constant Gen0 GC pressure. `ArrayPool<byte>.Shared` maintains reusable arrays — no allocation, no GC.

**Keywords:** heap allocation, GC pressure, rent/return, Gen0, hot path optimization

---

## SECTION 7 — Design Patterns & Architecture (Q73–Q82)

---

### Q73. What is CQRS and why use it?

**Strong Answer**
CQRS separates reads (queries) from writes (commands) — they can be optimized independently. Commands mutate state; queries return data with no side effects.

```csharp
public record CreateOrderCommand(int CustomerId, List<OrderItem> Items);
public record GetOrderQuery(int OrderId);
```

Often implemented with MediatR.

**Keywords:** command, query, MediatR, separation of concerns, read/write optimization

---

### Q74. What is the Outbox pattern and why do you need it?

**Strong Answer**
Ensures a DB write and message publish happen atomically. Write the event to an `Outbox` table in the same transaction as the domain change — a background job reads and publishes to the broker.

```
Without Outbox: Save order ✅ → Publish event → broker crash ❌ → inconsistency
With Outbox:    Save order + OutboxMessage in one transaction ✅ → background job publishes ✅
```

**Keywords:** at-least-once delivery, transactional outbox, eventual consistency, idempotency

---

### Q75. What is the Saga pattern?

**Strong Answer**
Saga manages long-running distributed transactions — each step publishes an event, next service reacts. On failure, compensating transactions roll back prior steps.

```
OrderCreated → PaymentProcessed → StockReserved
               PaymentFailed → OrderCancelled (compensating)
```

**Keywords:** distributed transaction, choreography vs orchestration, compensating transaction, eventual consistency

---

### Q76. What is Clean Architecture?

**Strong Answer**
Organizes code into concentric layers — inner layers have no dependency on outer layers. Domain at center, infrastructure at outside. Dependencies point inward via interfaces.

```
Domain → Application → Infrastructure → Presentation
```

**Keywords:** dependency inversion, domain-centric, use cases, ports and adapters, testability

---

### Q77. What is the Decorator pattern?

**Strong Answer**
Wraps an object to add behavior without modifying it — implements the same interface, delegates to the inner, adds logic around it.

```csharp
public class CachedOrderRepository : IOrderRepository
{
    private readonly IOrderRepository _inner;
    private readonly IMemoryCache _cache;

    public async Task<Order> GetByIdAsync(int id) =>
        await _cache.GetOrCreateAsync($"order:{id}", _ => _inner.GetByIdAsync(id));
}
```

**Keywords:** wrapping, same interface, cross-cutting concern, open/closed, Scrutor

---

### Q78. What is the Mediator pattern and how does MediatR implement it?

**Strong Answer**
Mediator centralizes communication — components send requests through a mediator instead of calling each other. MediatR uses `IRequest<T>` and `IRequestHandler<T>`, decoupling controllers from services.

**Keywords:** decoupling, IRequest, IRequestHandler, pipeline behavior, CQRS enabler

---

### Q79. What is event sourcing?

**Strong Answer**
Stores state as a sequence of events — current state is rebuilt by replaying all events. Provides full audit log and time-travel queries.

```
OrderCreated → PaymentReceived → OrderShipped
Current state = replay of all events
```

**Keywords:** append-only, event store, replay, audit log, eventual consistency, projection

---

### Q80. What is the difference between domain events and integration events?

**Strong Answer**
Domain events — same bounded context, same process, same transaction, synchronous. Integration events — cross service boundaries, published to message broker, asynchronous.

**Keywords:** bounded context, same transaction, message broker, async, at-least-once, idempotency

---

### Q81. What is a circuit breaker pattern?

**Strong Answer**
Prevents cascading failures — after N failures the circuit "opens", subsequent calls fail fast. After a timeout, "half-open" allows one probe request.

```csharp
var policy = Policy
    .HandleResult<HttpResponseMessage>(r => !r.IsSuccessStatusCode)
    .CircuitBreakerAsync(3, TimeSpan.FromSeconds(30));
```

**Keywords:** fail fast, cascading failure, open/closed/half-open, Polly, resilience

---

### Q82. What is the difference between horizontal and vertical scaling?

**Strong Answer**
Vertical — add CPU/RAM to existing server. Simple but has limits. Horizontal — add more instances behind a load balancer. Stateless services scale horizontally easily — stateful services need distributed state (Redis).

**Keywords:** scale-up vs scale-out, load balancer, stateless, distributed cache, Kubernetes

---

## SECTION 8 — Testing (Q83–Q88)

---

### Q83. What is the difference between unit, integration, and end-to-end tests?

**Strong Answer**
- **Unit** — single class, dependencies mocked, no I/O. Fast.
- **Integration** — multiple components together, real I/O. Slower.
- **End-to-end** — full system from UI to DB. Slowest, most fragile.

**Keywords:** test pyramid, isolation, mocking, xUnit, Moq, WebApplicationFactory

---

### Q84. What is mocking and how do you use Moq?

**Strong Answer**
Mocking replaces real dependencies with controllable fakes for unit isolation.

```csharp
var mockEmail = new Mock<IEmailService>();
mockEmail.Setup(s => s.SendAsync(It.IsAny<string>(), It.IsAny<string>()))
         .ReturnsAsync(true);

var service = new OrderService(mockEmail.Object);
await service.PlaceOrderAsync(order);

mockEmail.Verify(s => s.SendAsync(order.CustomerEmail, It.IsAny<string>()), Times.Once);
```

**Keywords:** Setup, Returns, Verify, It.IsAny, Times, mock vs stub vs fake

---

### Q85. What is `WebApplicationFactory` and when do you use it?

**Strong Answer**
Spins up a real ASP.NET Core app in memory for integration tests. Swap services (in-memory DB) to keep tests fast and isolated.

```csharp
public class OrderApiTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly HttpClient _client;
    public OrderApiTests(WebApplicationFactory<Program> factory)
    {
        _client = factory.WithWebHostBuilder(b =>
            b.ConfigureServices(s =>
                s.AddDbContext<AppDbContext>(o => o.UseInMemoryDatabase("TestDb"))))
            .CreateClient();
    }
}
```

**Keywords:** in-memory HTTP pipeline, service override, IClassFixture, integration testing

---

### Q86. What is the Arrange-Act-Assert pattern?

**Strong Answer**
AAA separates setup, execution, and verification into three clear sections.

```csharp
[Fact]
public async Task PlaceOrder_ShouldSendEmail()
{
    // Arrange
    var mockEmail = new Mock<IEmailService>();
    var service = new OrderService(mockEmail.Object);

    // Act
    await service.PlaceOrderAsync(order);

    // Assert
    mockEmail.Verify(e => e.SendAsync(order.CustomerEmail, It.IsAny<string>()), Times.Once);
}
```

**Keywords:** readability, single responsibility per test, Given-When-Then

---

### Q87. What is code coverage and is 100% a good goal?

**Strong Answer**
Coverage measures what percentage of code is executed by tests. 100% is not a good goal — trivial code (getters, constructors) can be covered meaninglessly. Aim for high coverage of business logic, edge cases, and error paths. Mutation testing is a better quality indicator.

**Keywords:** line coverage, branch coverage, mutation testing, meaningful assertions, diminishing returns

---

### Q88. What is the difference between `xUnit`, `NUnit`, and `MSTest`?

**Strong Answer**
xUnit is the modern standard — used by ASP.NET Core itself. `[Fact]` for single test, `[Theory]` + `[InlineData]` for parameterized, constructor for setup.

```csharp
[Theory]
[InlineData(1, 1, 2)]
[InlineData(2, 3, 5)]
public void Add_ShouldReturnSum(int a, int b, int expected)
    => Assert.Equal(expected, Calculator.Add(a, b));
```

**Keywords:** xUnit, [Fact], [Theory], [InlineData], IClassFixture, constructor injection

---

## SECTION 9 — Git, DevOps & Tooling (Q89–Q95)

---

### Q89. What is the difference between `git merge` and `git rebase`?

**Strong Answer**
`merge` — creates merge commit, preserves full history. `rebase` — replays commits on top of another branch, linear history. Never rebase shared/public branches — rewrites history.

```bash
git merge feature/payments     # merge commit
git rebase main                # linear history, rewrites commits
```

**Keywords:** merge commit, linear history, rewrite history, golden rule of rebase, fast-forward

---

### Q90. What is a pull request and what makes a good code review?

**Strong Answer**
A PR proposes changes from a branch for review before merging. Good review checks: correctness, test coverage, edge cases, security (injection, auth), performance (N+1, missing index), code conventions. Comments should be constructive — explain why, suggest alternatives.

**Keywords:** code review, branch protection, CI gates, constructive feedback, nitpick vs blocker

---

### Q91. What is CI/CD?

**Strong Answer**
CI — automatically builds and tests on every push. CD — automatically deploys passing builds to staging/production.

```
Push → Build → Unit Tests → Integration Tests → Deploy to staging → Deploy to prod
```

**Keywords:** build pipeline, automated tests, deployment automation, GitHub Actions, Azure DevOps, rollback

---

### Q92. What is Docker and why is it useful?

**Strong Answer**
Docker packages an app and dependencies into a container — runs the same everywhere. Eliminates "works on my machine". In .NET, produces a self-contained image deployable anywhere Docker runs.

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app
COPY --from=build /app/publish .
ENTRYPOINT ["dotnet", "MyApi.dll"]
```

**Keywords:** container, image, Dockerfile, docker-compose, environment parity, registry, Kubernetes

---

### Q93. What is the difference between `git reset` and `git revert`?

**Strong Answer**
`git reset` moves the branch pointer backward — rewrites history, dangerous on shared branches. `git revert` creates a new "undo" commit — safe, history preserved.

```bash
git reset --hard HEAD~1  # removes commit from history
git revert abc1234       # adds new undo commit
```

**Keywords:** history rewrite, safe undo, shared branch, hard vs soft reset

---

### Q94. What is semantic versioning (SemVer)?

**Strong Answer**
`MAJOR.MINOR.PATCH`:
- MAJOR — breaking changes
- MINOR — new backward-compatible features
- PATCH — bug fixes

**Keywords:** breaking change, backward compatibility, NuGet versioning

---

### Q95. What is the purpose of `appsettings.json` and environment overrides?

**Strong Answer**
`appsettings.json` is base config. `appsettings.{Environment}.json` overrides per environment. Sensitive values come from environment variables or secrets manager — never committed to source control.

```csharp
var connStr = configuration.GetConnectionString("Default");
```

**Keywords:** configuration hierarchy, environment variables, user secrets, IConfiguration, 12-factor app

---

## SECTION 10 — Mixed Practical (Q96–Q100)

---

### Q96. What is the difference between `Equals` override and `IEquatable<T>`?

**Strong Answer**
`object.Equals` is untyped — requires casting. `IEquatable<T>` is typed, allocation-free. Implement both for value objects — always override `GetHashCode` when overriding `Equals`.

```csharp
public class Money : IEquatable<Money>
{
    public bool Equals(Money other) =>
        other != null && Amount == other.Amount && Currency == other.Currency;

    public override bool Equals(object obj) => Equals(obj as Money);
    public override int GetHashCode() => HashCode.Combine(Amount, Currency);
}
```

**Keywords:** GetHashCode contract, value object, IEquatable, boxing avoidance, HashCode.Combine

---

### Q97. What causes `NullReferenceException` and how do you prevent it in modern C#?

**Strong Answer**
Dereferencing a null reference. Modern C# prevents with nullable reference types (`#nullable enable`) — compiler warns on potential null dereferences. Combine with null-conditional and guard clauses.

```csharp
#nullable enable
public string GetCity(User? user)
{
    ArgumentNullException.ThrowIfNull(user);
    return user?.Address?.City ?? "Unknown";
}
```

**Keywords:** nullable reference types, #nullable enable, null-conditional, guard clause, ArgumentNullException.ThrowIfNull

---

### Q98. What is the difference between `Dictionary` and `ConcurrentDictionary`?

**Strong Answer**
`Dictionary` is not thread-safe — concurrent writes cause corruption. `ConcurrentDictionary` uses fine-grained locking for writes, lock-free reads, atomic `GetOrAdd`/`AddOrUpdate`.

```csharp
var concurrent = new ConcurrentDictionary<string, int>();
concurrent.GetOrAdd("key", _ => ComputeValue());
concurrent.AddOrUpdate("key", 1, (k, existing) => existing + 1);
```

**Keywords:** thread safety, fine-grained locking, lock-free reads, GetOrAdd atomicity, race condition

---

### Q99. What is `Lazy<T>` and when do you use it?

**Strong Answer**
`Lazy<T>` defers creation until first access — thread-safe by default. Use for expensive initialization that may not always be needed.

```csharp
private readonly Lazy<ExpensiveService> _service =
    new Lazy<ExpensiveService>(() => new ExpensiveService());

public void DoWork() => _service.Value.Process();
```

**Keywords:** deferred initialization, thread-safe, factory function, LazyThreadSafetyMode, singleton initialization

---

### Q100. You have a memory leak in production — walk me through how you diagnose it.

**Strong Answer**
1. **Confirm** — `dotnet-counters monitor` — watch `gc-heap-size` growing without releasing.
2. **Capture dump** — `dotnet-dump collect -p <pid>` at peak memory.
3. **Analyze** — `dotnet-dump analyze` → `dumpheap -stat` — find types consuming most memory.
4. **Find GC roots** — `gcroot <address>` — shows what keeps objects alive.
5. **Common culprits** — static collections, event handlers not unsubscribed, unbounded caches, DbContext not disposed.
6. **Fix and verify** — deploy, monitor memory stabilizes.

```bash
dotnet-counters monitor --process-id 1234 \
  --counters System.Runtime[gc-heap-size,gen-0-gc-count]

dotnet-dump collect -p 1234
dotnet-dump analyze ./core_dump
> dumpheap -stat
> gcroot 00007f1234abcd
```

**Keywords:** dotnet-counters, dotnet-dump, dumpheap, gcroot, GC roots, event handler leak, heap analysis

---

## Quick Reference — All Keywords by Section

```
C# Fundamentals:    stack/heap, boxing, immutable string, StringBuilder,
                    value equality, null safety, readonly vs const,
                    generics, yield return, IDisposable, delegates

OOP & Design:       SOLID, DI, IoC container, lifetimes, Repository,
                    Unit of Work, composition over inheritance,
                    Factory, Strategy, Observer, Decorator

Async/Concurrency:  async/await, state machine, Task, CancellationToken,
                    SemaphoreSlim, Parallel.ForEachAsync, IAsyncEnumerable,
                    deadlock, ConfigureAwait, Channel<T>, lock

EF Core:            IQueryable, AsNoTracking, N+1, Include, migrations,
                    SaveChanges, ExecuteUpdateAsync, transactions,
                    index, covering index, connection pooling

ASP.NET Core:       middleware, pipeline, JWT, authentication/authorization,
                    model binding, IActionResult, CORS, IHttpClientFactory,
                    IMemoryCache, HTTP status codes

Memory/Perf:        GC generations, Span<T>, ArrayPool, object pooling,
                    struct vs class, record, memory leak diagnosis

Architecture:       CQRS, Outbox, Saga, Clean Architecture, Mediator,
                    event sourcing, domain vs integration events,
                    circuit breaker, horizontal vs vertical scaling

Testing:            unit/integration/e2e, Moq, WebApplicationFactory,
                    AAA pattern, xUnit, code coverage

DevOps:             CI/CD, Docker, git merge vs rebase, SemVer,
                    appsettings hierarchy, environment variables

Practical:          IEquatable, NullReferenceException, Dictionary vs
                    ConcurrentDictionary, Lazy<T>, memory leak diagnosis
```
