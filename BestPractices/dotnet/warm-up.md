# .NET / ASP.NET Interview Review Notes

## 1. Abstract class vs Interface

**Strong answer**

An abstract class is a base class used when multiple related classes
share common state or behavior. It can contain fields, constructors,
implemented methods, and abstract methods that derived classes must
implement.

An interface mainly defines a contract of behaviors a class must
provide. A class can implement multiple interfaces, but inherit only one
base or abstract class.

**Keywords interviewer wants to hear** - Shared behavior - Contract -
Fields/state - Constructors - Multiple interfaces - Single inheritance

------------------------------------------------------------------------

## 2. Task vs Thread

**Strong answer**

A Thread is an actual OS execution thread that runs code. A Task
represents an asynchronous unit of work and is usually scheduled on
threads from the ThreadPool. Tasks are higher-level and work well with
async/await.

**Keywords interviewer wants to hear** - Asynchronous operation -
ThreadPool - OS thread - async/await - Higher-level abstraction

------------------------------------------------------------------------

## 3. Async void vs Async Task

**Strong answer**

async Task returns a Task so the caller can await it and handle
exceptions properly. async void cannot be awaited and exceptions are
difficult to handle. async void is mainly used for event handlers.

**Keywords interviewer wants to hear** - Await - Exception handling -
Event handlers - Recommended practice

------------------------------------------------------------------------

## 4. AddTransient vs AddScoped vs AddSingleton

**Strong answer**

AddTransient creates a new instance every time the service is resolved.

AddScoped creates one instance per HTTP request.

AddSingleton creates one instance for the entire application lifetime.

**Keywords interviewer wants to hear** - Service lifetime - Dependency
Injection - Resolved - HTTP request - Application lifetime

------------------------------------------------------------------------

## 5. Middleware

**Strong answer**

Middleware is a component in the ASP.NET Core request pipeline that
processes HTTP requests and responses. Middleware can inspect, modify,
stop, or pass requests to the next middleware.

**Keywords interviewer wants to hear** - Request pipeline -
Request/response - Next middleware - Inspect - Modify

------------------------------------------------------------------------

## 6. String vs StringBuilder

**Strong answer**

string is immutable, meaning modifications create a new object.
StringBuilder is mutable and efficiently modifies content without
creating many new objects.

Use string for simple operations and StringBuilder for frequent
modifications.

**Keywords interviewer wants to hear** - Immutable - Mutable -
Performance - Memory allocation

------------------------------------------------------------------------

## 7. FirstOrDefault vs SingleOrDefault

**Strong answer**

FirstOrDefault returns the first matching element or default value if
none exists.

SingleOrDefault expects at most one matching result and throws an
exception if multiple results exist.

**Keywords interviewer wants to hear** - Unique data - Exception - First
match - Default value

------------------------------------------------------------------------

## 8. Dependency Injection

**Strong answer**

Dependency Injection is a design pattern where dependencies are provided
from outside rather than being created inside the class. It reduces
coupling, improves maintainability, and makes testing easier.

**Keywords interviewer wants to hear** - Loose coupling -
Maintainability - Testing - Separation of concerns

------------------------------------------------------------------------

## 9. IEnumerable vs IQueryable

**Strong answer**

IEnumerable performs operations in memory.

IQueryable builds an expression tree that can be translated into SQL and
executed in the database.

**Keywords interviewer wants to hear** - Expression tree - Deferred
execution - SQL translation - In-memory - Performance

------------------------------------------------------------------------

## 10. DbContext

**Strong answer**

DbContext represents a session between the application and database. It
provides access through DbSet, tracks entity changes, and manages CRUD
operations.

**Keywords interviewer wants to hear** - Session - DbSet - Change
tracking - CRUD

------------------------------------------------------------------------

## 11. SaveChanges()

**Strong answer**

SaveChanges checks the change tracker, identifies entity states such as
Added, Modified, and Deleted, generates SQL statements, and executes
them against the database.

**Keywords interviewer wants to hear** - Change tracker - Entity
states - SQL generation - Added - Modified - Deleted

------------------------------------------------------------------------

## 12. Eager vs Lazy vs Explicit Loading

**Strong answer**

Eager loading loads related entities immediately using Include().

Lazy loading automatically loads related entities when accessed.

Explicit loading manually loads related entities when needed.

**Keywords interviewer wants to hear** - Include() - N+1 problem -
Related entities - Manual loading
