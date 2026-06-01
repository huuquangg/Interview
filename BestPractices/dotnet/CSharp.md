
## **"Can you explain the difference between a value type and a reference type in C#? Give me an example of each."**

_"Value types are stored on the **stack** and when assigned or passed, a **copy** is made — so the original is not affected. Examples: `int`, `bool`, `struct`._

_Reference types are stored on the **heap** — the variable holds a **pointer** to the object. When you assign or pass it, you're sharing the same reference. So modifying it anywhere affects the same object. Examples: `class`, `string`, `array`."_
- Core:
	- make a copy when assigned or passed -> value not modify
	- sharing the same ref when assign or passed -> value is modified
- Mention: stack and heap; pointer

---

## "What is the difference between `interface` and `abstract class` in C#? When would you choose one over the other?"

_"An interface defines a **contract** — a list of method/property signatures with no implementation. A class can implement **multiple** interfaces. An abstract class can have both implemented and abstract methods, constructors, and fields — but a class can only inherit **one**._

_I'd choose an abstract class when multiple classes share common behavior — like a `BaseAgent` with shared logging logic. I'd choose an interface when I just want to enforce a capability — like `IBookable` or `IDisposable`."_
- Core:
	- method/property signatures with no implementation (interfaces)
	- implemented and abstract methods, constructors, and fields
- Mention: contract, inherit one

---
## Can you explain `async` and `await` in C#? And what is the difference between `async Task` and `async void`?"
_"`async` marks a method as asynchronous. `await` suspends the method and **frees the thread** until the Task completes — it's non-blocking, unlike `.Wait()` which blocks._

_`async Task` is awaitable and exceptions propagate correctly — always prefer this. `async void` cannot be awaited, and if it throws an exception, it's unhandled and can crash the application. The only valid use for `async void` is event handlers."_
- Core:
	- `async` marks a method as asynchronous
	- `await` suspends the method and **frees the thread** until the Task completes
	- _`async Task` is awaitable and exceptions
	- `async void` cannot be awaited, and if it throws an exception, it's unhandled and can crash the application.
- Mention: non-blocking

---
## "Can you explain what **LINQ** is? What is the difference between `IEnumerable` and `IQueryable`, and why does it matter in Entity Framework?"
_"LINQ stands for Language Integrated Query — it's a C# feature that lets you query data using familiar C# syntax instead of raw SQL. Entity Framework uses LINQ to generate SQL queries._

_`IQueryable` builds an **expression tree** that EF Core translates to SQL — so filtering happens **in the database**. `IEnumerable` pulls **all data into memory first**, then filters in C#. For large tables this is a huge performance problem._

_So in EF Core I always keep queries as `IQueryable` and only call `.ToListAsync()` at the very end."_

---
## "What is the **Repository Pattern**? Why would you use it instead of calling `DbContext` directly in your controller or service?"

_"The Repository Pattern abstracts data access behind an interface. Instead of my service calling `DbContext` directly, it calls `IBookingRepository`. The concrete implementation handles the EF Core logic._

_The reasons I'd use it over direct `DbContext`:_

- **Testability** — I can mock `IBookingRepository` in unit tests without needing a real database
- **Separation of concerns** — business logic stays clean, data access stays in one place
- **Flexibility** — I can swap EF Core for Dapper or a different database without touching the service layer"*

---
## _"Give me a scenario: your API endpoint is taking **5 seconds** to respond in production. Walk me through how you would investigate and fix it."_

"First I'd add timing logs or use a profiler to isolate exactly where the 5 seconds is spent — is it the code, the database, or an external service call?
If it's the database:

Check if query uses IQueryable with proper filters, not IEnumerable
Look for N+1 queries — use .Include() or projection
Add AsNoTracking() for read-only queries
Check if indexed columns are used in WHERE clauses
Add pagination if returning large datasets

If it's the code:

Check if multiple async calls are sequential — use Task.WhenAll() to parallelize
Chunk large write operations into batches

If it's an external service:

Add timeout + retry policy (Polly library)
Consider caching the response with IMemoryCache"*

---