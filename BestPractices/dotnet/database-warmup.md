# .NET Large Data Management — Interview Q&A

> **Target level:** Junior → entry Mid-level .NET Software Engineer
> **Topics:** EF Core, IQueryable, Pagination, Streaming, Concurrency, Background Jobs

---

## 1. IQueryable vs IEnumerable

**Q: What is the difference between `IQueryable` and `IEnumerable`, and why does it matter for large data?**

### Strong Answer
`IQueryable` keeps the query as an expression tree that EF Core translates into SQL — filtering happens inside the database. `IEnumerable` executes in C# memory — all rows are loaded first, then filtered in the application process.

For large data this is critical: a misplaced `.ToList()` before a `.Where()` can load millions of rows into RAM when the database could have returned only a handful.

```csharp
// BAD — loads all orders into memory, then filters in C#
var results = dbContext.Orders
    .Where(o => o.CustomerId == customerId)
    .ToList()                           // IQueryable → IEnumerable boundary
    .Where(o => o.Total > 100);         // runs in C#

// GOOD — single SQL with both filters
var results = dbContext.Orders
    .AsNoTracking()
    .Where(o => o.CustomerId == customerId && o.Total > 100)
    .ToList();
```

### Keywords interviewer wants to hear
- **Expression tree** — IQueryable builds one; EF Core translates it to SQL
- **SQL boundary** — the point where IQueryable becomes IEnumerable
- **Client evaluation** — when filtering moves from DB to C# (bad for large data)
- **AsNoTracking** — skips the EF change tracker for read-only queries; reduces memory overhead

---

## 2. AsNoTracking

**Q: When should you use `AsNoTracking()` and what does it actually do?**

### Strong Answer
EF Core's change tracker watches every entity it loads so it can detect modifications on `SaveChanges()`. For read-only queries this is wasted overhead — memory and CPU spent tracking objects you never intend to update.

`AsNoTracking()` tells EF to skip tracking entirely. The entity is returned as a plain object with no snapshot stored.

```csharp
// Without AsNoTracking — EF tracks every Product in memory
var products = dbContext.Products.ToList();

// With AsNoTracking — lighter, faster, no tracking overhead
var products = dbContext.Products
    .AsNoTracking()
    .ToList();
```

Use `AsNoTracking` for: API read endpoints, reports, exports, any query where you don't call `SaveChanges()` afterward.

Do NOT use for: queries where you load an entity and then update it in the same DbContext scope.

### Keywords interviewer wants to hear
- **Change tracker** — what AsNoTracking disables
- **Read-only query** — the use case
- **Memory overhead** — what you save
- **Identity resolution** — also skipped with AsNoTracking (EF won't deduplicate related entities)

---

## 3. `.ToList()` vs `.AsEnumerable()` vs `.AsAsyncEnumerable()`

**Q: What is the difference between these three, and when do you use each?**

### Strong Answer

| Method | Executes immediately | Loads all to RAM | Async | DB connection |
|---|---|---|---|---|
| `.ToList()` | Yes | Yes | No (use ToListAsync) | Closes after load |
| `.AsEnumerable()` | Lazy | No — row by row | No | Stays open during iteration |
| `.AsAsyncEnumerable()` | Lazy | No — row by row | Yes | Stays open during iteration |

```csharp
// ToList — small/medium result sets, need all data in RAM
var products = await dbContext.Products
    .Where(p => p.CategoryId == 5)
    .AsNoTracking()
    .ToListAsync();

// AsEnumerable — need C# logic EF can't translate to SQL
var results = dbContext.Products
    .Where(p => p.CategoryId == 5)   // SQL filter first
    .AsEnumerable()                   // switch to memory
    .Where(p => MyCustomRule(p));     // C# logic EF can't translate

// AsAsyncEnumerable — large exports, streaming responses
await foreach (var product in dbContext.Products.AsAsyncEnumerable())
{
    await writer.WriteLineAsync(product.Name); // one row in memory at a time
}
```

**Filter first, switch late** — always push `Where`, `Select`, `OrderBy` above `.AsEnumerable()` so the DB does the heavy lifting before rows come across the wire.

### Keywords interviewer wants to hear
- **Lazy evaluation** — AsEnumerable / AsAsyncEnumerable don't execute until iterated
- **DB connection stays open** — trade-off of streaming; process rows quickly
- **EF translation exception** — reason to use AsEnumerable for untranslatable logic
- **Filter first, switch late** — discipline for query chain structure
- **IAsyncEnumerable** — the return type for async streaming

---

## 4. Offset vs Keyset Pagination

**Q: You implement pagination with `.Skip().Take()`. It works on page 1 but is very slow on page 400,000. Why, and how do you fix it?**

### Strong Answer
`Skip(n)` translates to SQL `OFFSET n ROWS`. The database must scan and count through every skipped row to find where the requested page starts. Performance degrades linearly — O(n) — the deeper the page, the slower the query.

```sql
-- Offset: DB counts through 40,000,000 rows before returning anything
SELECT * FROM Orders ORDER BY Id
OFFSET 40000000 ROWS FETCH NEXT 100 ROWS ONLY
```

The fix is **keyset pagination** — instead of skipping rows, anchor to the last seen ID and use the database index to seek directly:

```csharp
// Offset — slow at depth
var page = dbContext.Orders
    .OrderBy(o => o.Id)
    .Skip(pageNumber * pageSize)
    .Take(pageSize)
    .ToList();

// Keyset — always fast
var page = dbContext.Orders
    .Where(o => o.Id > lastSeenId)   // seek via index
    .OrderBy(o => o.Id)
    .Take(pageSize)
    .ToListAsync();

// Save cursor for next page
lastSeenId = page.Last().Id;
```

The DB uses the **B-tree index on Id** to jump directly to `lastSeenId` in O(log n) time — no scanning, no counting.

**Trade-off:** keyset cannot jump to an arbitrary page number. You can only move forward sequentially using the cursor. For most feeds, lists, and exports this is acceptable.

**Important:** keyset is only fast if the column you paginate by is **indexed**. Primary keys are indexed automatically by the database. Non-PK columns (e.g. `CreatedAt`) require a manual index:

```csharp
// Fluent API — add index for non-PK keyset column
modelBuilder.Entity<Order>()
    .HasIndex(o => o.CreatedAt);
```

### Keywords interviewer wants to hear
- **OFFSET degradation** — O(n) scan cost at deep pages
- **Keyset / cursor-based pagination** — the solution
- **B-tree index seek** — why keyset is O(log n)
- **Cursor** — the lastSeenId passed between pages
- **Primary key auto-indexed** — PK index is created by DB automatically
- **Trade-off: no arbitrary page jump** — honest acknowledgment of the limitation

---

## 5. Streaming Large Data

**Q: A junior wrote an endpoint that calls `.ToListAsync()` on 2 million rows to generate a CSV export. What is wrong and how do you fix it?**

### Strong Answer
`.ToListAsync()` materializes all 2 million rows into a `List<T>` in RAM simultaneously. This risks out-of-memory exceptions, causes GC pressure, and blocks the response until all rows are loaded.

The fix is to stream rows using `AsAsyncEnumerable()` — only one row is in memory at a time, and the response starts writing immediately:

```csharp
// BAD — 2M rows loaded to RAM at once
var all = await dbContext.Reports.ToListAsync();
foreach (var row in all) { /* write CSV */ }

// GOOD — one row in memory at a time
var stream = dbContext.Reports
    .AsNoTracking()
    .AsAsyncEnumerable();

await foreach (var row in stream)
{
    await writer.WriteLineAsync(FormatCsv(row));
}
```

For very large exports, also consider chunking with `.Chunk(1000)` if you need to close the DB connection between batches or do parallel processing per batch.

### Keywords interviewer wants to hear
- **Memory exhaustion / OOM** — the risk of ToList on large sets
- **AsAsyncEnumerable** — the streaming solution
- **GC pressure** — secondary cost of large allocations
- **Chunk(n)** — batched alternative when connection must close between batches
- **DB connection stays open** — trade-off of AsAsyncEnumerable streaming

---

## 6. Unbounded Concurrency

**Q: What is wrong with this code when `alerts` has 10,000 items?**

```csharp
var tasks = alerts.Select(a => ProcessAlertAsync(a));
await Task.WhenAll(tasks);
```

### Strong Answer
`.Select()` enumerates immediately — `ProcessAlertAsync` is called for all 10,000 items before `Task.WhenAll` starts waiting. This spawns 10,000 concurrent tasks simultaneously, which can exhaust the database connection pool, overwhelm downstream services, and crash the process.

The fix is **bounded concurrency** using `SemaphoreSlim`:

```csharp
// SemaphoreSlim — manual bounded concurrency
var semaphore = new SemaphoreSlim(10); // max 10 concurrent

var tasks = alerts.Select(async a =>
{
    await semaphore.WaitAsync();
    try { await ProcessAlertAsync(a); }
    finally { semaphore.Release(); }
});

await Task.WhenAll(tasks);
```

Or more cleanly in .NET 6+ using `Parallel.ForEachAsync`:

```csharp
// Parallel.ForEachAsync — built-in bounded concurrency
await Parallel.ForEachAsync(alerts,
    new ParallelOptions { MaxDegreeOfParallelism = 10 },
    async (alert, ct) => await ProcessAlertAsync(alert));
```

`Parallel.ForEachAsync` is preferred for large collections — cleaner syntax, built-in cancellation token support, no manual semaphore management.

### Keywords interviewer wants to hear
- **Connection pool exhaustion** — the consequence of unbounded concurrency
- **SemaphoreSlim** — the manual bounded concurrency primitive
- **MaxDegreeOfParallelism** — the Parallel.ForEachAsync equivalent
- **Parallel.ForEachAsync** — .NET 6+ preferred solution
- **Task.WhenAll vs Parallel.ForEachAsync** — know the difference and when to use each

---

## 7. Long-Running Job Architecture

**Q: A report generation job takes 2–3 minutes. A junior put it directly in an API endpoint. What are the problems and how do you redesign it?**

### Strong Answer
Three problems:

1. **Memory** — `ToListAsync()` on 1M rows loads everything into RAM at once
2. **HTTP timeout** — gateways, load balancers, and clients time out long before 2–3 minutes
3. **Blocking the request thread** — the API thread is held for the entire job duration, reducing throughput

The correct design is an **async job pattern with 202 Accepted**:

```
POST /generate-report
→ validate request
→ enqueue job to background queue
→ return 202 Accepted + jobId immediately

// BackgroundService consumes the queue:
→ stream 1M rows with AsAsyncEnumerable + AsNoTracking
→ process and write file in chunks
→ checkpoint progress (persist lastProcessedId)
→ mark job complete in DB

GET /report-status/{jobId}
→ return { status: "pending" | "complete" | "failed", fileUrl }
```

```csharp
// Controller — returns immediately
[HttpPost("generate-report")]
public async Task<IActionResult> GenerateReport()
{
    var jobId = Guid.NewGuid();
    await _queue.EnqueueAsync(jobId);
    return Accepted(new { jobId });   // 202 Accepted
}

// BackgroundService — processes asynchronously
await foreach (var row in dbContext.Records.AsAsyncEnumerable())
{
    await ProcessRowAsync(row);
}
```

### Keywords interviewer wants to hear
- **202 Accepted** — the correct HTTP status for async jobs
- **BackgroundService / IHostedService** — .NET background processing host
- **Decouple from HTTP request** — the core architectural principle
- **Queue** — RabbitMQ, Azure Service Bus, or Channel\<T\> for in-process
- **Checkpointing** — persist lastProcessedId so job resumes after crash
- **Polling** — client polls `/report-status/{jobId}` for completion

---

## Quick Reference — Keywords Cheat Sheet

```
IQueryable      → SQL translation, expression tree, DB filtering
IEnumerable     → C# memory, client evaluation
AsNoTracking    → skip change tracker, read-only queries
AsAsyncEnumerable → async streaming, one row at a time
ToList          → materializes all rows to RAM immediately

Offset pagination   → OFFSET/FETCH, O(n) degradation at depth
Keyset pagination   → WHERE id > lastId, O(log n) index seek, cursor
B-tree index        → enables fast keyset seek
Primary key         → auto-indexed by DB, no annotation needed
HasIndex()          → Fluent API for non-PK index

SemaphoreSlim           → manual bounded concurrency
Parallel.ForEachAsync   → .NET 6+ built-in bounded concurrency
MaxDegreeOfParallelism  → concurrency limit setting
Connection pool exhaustion → consequence of unbounded Task.WhenAll

202 Accepted    → async job accepted, poll for status
BackgroundService → IHostedService, queue consumer
Checkpointing   → persist progress for crash recovery
Chunk(n)        → batch rows, close connection between batches
```
