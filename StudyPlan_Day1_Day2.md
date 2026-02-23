# 📅 Study Plan — Day 1 & Day 2

> **Theory-focused** revision guide. Less code, more explanations.
> Cross-reference with module files for deeper dives.

---

# ☀️ DAY 1 — C# + OOP + .NET Basics

> **Goal:** Build a strong base for ASP.NET Core & EF Core.

---

## 1. OOP — The Four Pillars

### 🔷 Encapsulation
> **Hiding internal state and requiring all interaction to go through well-defined methods/properties.**

- Bundle data (fields) and behavior (methods) together in a class
- Use **access modifiers** to control visibility: `private`, `protected`, `internal`, `public`
- Expose data through **properties** (get/set), not raw fields
- **Why?** Prevents outside code from putting an object into an invalid state

> *Example:* A `BankAccount` class keeps `balance` private. Outside code can only call `Deposit()` or `Withdraw()` — it cannot directly set `balance = -1000`.

---

### 🔷 Abstraction
> **Showing only what is necessary and hiding implementation complexity.**

- Users of a class see only the **interface/contract**, not how it works internally
- Achieved via **abstract classes** and **interfaces**
- **Why?** Reduces complexity, makes code easier to use and change

> *Example:* When you call `dbContext.SaveChangesAsync()`, you don't need to know that it generates SQL, tracks changes, handles concurrency. You just call it.

---

### 🔷 Inheritance
> **A class (child) acquires properties and behavior from another class (parent).**

- Use `extends` keyword (`:` in C#)
- Child class gets all non-private members of parent
- Child can **override** virtual/abstract methods
- **Why?** Code reuse — write shared logic once in parent

> *Example:* `EmployeeController : ControllerBase` — your controller inherits all HTTP response helper methods (`Ok()`, `NotFound()`, `BadRequest()`) from `ControllerBase`.

**Single Inheritance only** in C#. Use interfaces for multiple "inheritance."

---

### 🔷 Polymorphism
> **One interface, many implementations. The same method call behaves differently depending on the actual object type.**

Two types:
| Type | How | When resolved |
|---|---|---|
| **Compile-time** (Static) | Method overloading — same name, different params | At compile time |
| **Runtime** (Dynamic) | Method overriding — `virtual` + `override` | At runtime |

> *Example:* `Shape.Draw()` behaves differently when the actual object is a `Circle` vs a `Square`. The calling code doesn't need to know the type — it just calls `Draw()`.

---

## 2. Interfaces vs Abstract Class

| Aspect | Interface | Abstract Class |
|---|---|---|
| Instantiate | ❌ No | ❌ No |
| State (fields) | ❌ No | ✅ Yes |
| Constructor | ❌ No | ✅ Yes |
| Concrete methods | ✅ Default methods (C# 8+) | ✅ Yes |
| Multiple inheritance | ✅ A class can implement many | ❌ Only one base class |
| Relationship | "**can-do**" (ISwimmable, ILoggable) | "**is-a**" (Animal, Vehicle) |
| When to use | Define a **contract / capability** | Define a **shared base** with common logic |

> **Rule of thumb:**
> - Use **interface** when unrelated classes need a common contract (e.g., `ILogger` is implemented by `FileLogger`, `ConsoleLogger`, `SerilogLogger`)
> - Use **abstract class** when there is genuine shared state or behavior (e.g., `Animal` has a shared `Breathe()` method)

---

## 3. Value Types vs Reference Types

| Aspect | Value Type | Reference Type |
|---|---|---|
| Stored in | **Stack** | **Heap** |
| Examples | `int`, `float`, `bool`, `struct`, `enum` | `class`, `string`, `array`, `interface` |
| Assignment | Copies the **value** | Copies the **reference** (pointer) |
| Null | ❌ Cannot be null (unless `int?`) | ✅ Can be null |
| Default | `0`, `false`, etc. | `null` |

> **Key interview point:** When you assign `int a = b`, changing `a` does not change `b`. When you assign `Employee emp2 = emp1`, both point to the **same object** — changing `emp2.Name` also changes `emp1.Name`.

> `string` is a **reference type** but behaves like a value type because it is **immutable** — every "change" creates a new string object.

---

## 4. Delegates, Func, Action

### Delegate
> A **delegate** is a type-safe **function pointer** — a variable that holds a reference to a method.

- `delegate` keyword defines a delegate type
- Enables passing methods as parameters
- Foundation of **events** in C#

### Func and Action
> `Func` and `Action` are **built-in generic delegate types** — no need to define your own delegate.

| Type | Returns | Signature |
|---|---|---|
| `Action` | `void` | `Action<T>` — takes T, returns nothing |
| `Func` | Something | `Func<T, TResult>` — takes T, returns TResult |
| `Predicate<T>` | `bool` | Shorthand for `Func<T, bool>` |

> *Where you see them:*
> - `List.ForEach(Action<T>)` — pass a method that processes each item
> - `LINQ .Where(Func<T, bool>)` — pass a method that filters items
> - DI registrations, middleware, event handlers

---

## 5. Lambda Expressions
> A **lambda** is an anonymous (inline) function — a shorthand for creating delegates/Func/Action without writing a named method.

- Syntax: `parameters => expression`
- Used heavily with LINQ, events, and delegates
- Makes code concise and readable inline

> *Why important?* Almost all LINQ is written with lambdas: `employees.Where(e => e.Salary > 50000)`. Understanding lambdas = understanding LINQ.

---

## 6. async / await
> Enables **non-blocking** code for I/O operations (DB queries, HTTP calls, file reads).

**The key concept:** When `await` is reached, the current thread is **released back to the thread pool** while the I/O completes. When the I/O finishes, a thread picks up execution from where it left off.

**Why it matters for Web APIs:**
- Without async: Each request holds a thread for the entire duration of a DB call. With 100 threads in the pool, you handle ~100 concurrent requests.
- With async: Threads are released during I/O. The same 100 threads can handle **thousands** of concurrent requests.

**Key rules:**
| Rule | Detail |
|---|---|
| Return type | `Task` (void) or `Task<T>` (with return value) |
| Never `.Result` / `.Wait()` | Blocks the thread, causes deadlocks in ASP.NET Core |
| `async void` | Only for event handlers — no error propagation |
| Method naming | Suffix with `Async` (e.g., `GetEmployeeAsync`) |
| Parallel tasks | `await Task.WhenAll(task1, task2)` |

> See **Module 1 → Section 1.3** for code examples.

---

## 7. LINQ

> **Language Integrated Query** — query any collection (`List`, `IQueryable`, XML, etc.) using C# syntax.

**Common operators:**

| Operator | What it does |
|---|---|
| `Where` | Filter — returns elements matching a condition |
| `Select` | Project — transform each element into something else |
| `GroupBy` | Group elements by a key |
| `OrderBy` / `OrderByDescending` | Sort |
| `Join` | Combine two collections on a key |
| `FirstOrDefault` | Get first match, or null if none |
| `Any` / `All` | Check if any/all elements match a condition |
| `Count` | Count elements |
| `Sum` / `Average` / `Max` / `Min` | Aggregate |
| `ToList` / `ToArray` | Execute query, materialize results |
| `Include` | EF Core — eager load navigation properties |

**Deferred execution:** LINQ queries are **not executed** when defined — they execute when you iterate or call `.ToList()`, `.Count()`, etc. This is the foundation of `IQueryable` in EF Core.

---

## 8. Exception Handling

> Structured error handling using `try / catch / finally / throw`.

**Key concepts:**

| Concept | Detail |
|---|---|
| `try` | Code that might throw |
| `catch (ExceptionType ex)` | Handle specific exception type |
| `catch (Exception ex)` | Catch-all — use last, be careful |
| `finally` | Always runs — for cleanup (close connections, dispose) |
| `throw` | Re-throw current exception (preserves stack trace) |
| `throw ex` | Re-throw but **resets** stack trace — avoid |
| Custom exceptions | Inherit from `Exception` — add meaningful context |

**Global handling in ASP.NET Core:**
- Middleware (`UseExceptionHandler`) — catches unhandled exceptions pipeline-wide
- `IExceptionFilter` — Action Filter approach for controllers
- `ProblemDetails` — standardized error response format (RFC 7807)

> **Interview point:** Always catch the **most specific** exception first (before `Exception`). Use `finally` for resource cleanup instead of duplicating code in each catch block.

---

## 9. SOLID Principles

> Five design principles for writing **maintainable, extensible, testable** code.

| Letter | Principle | One-liner |
|---|---|---|
| **S** | Single Responsibility | A class should have **one reason to change** |
| **O** | Open/Closed | Open for **extension**, closed for **modification** |
| **L** | Liskov Substitution | Subtypes must be **substitutable** for their base type |
| **I** | Interface Segregation | Prefer **small, specific** interfaces over one large one |
| **D** | Dependency Inversion | Depend on **abstractions**, not concrete implementations |

**Why interviewers ask SOLID:**
- It shows you understand **why** code becomes hard to maintain
- Every SOLID violation has a concrete symptom (hard to test, huge merge conflicts, breaking changes)

> See **Module 2** for full details with examples.

---

## 10. .NET Core Basics

### .NET Core vs .NET Framework

| Aspect | .NET Framework | .NET Core / .NET 5+ |
|---|---|---|
| Platform | Windows only | Cross-platform (Windows, Linux, macOS) |
| Open source | ❌ | ✅ |
| Performance | Good | Significantly better |
| Deployment | Installed on machine | Can be self-contained |
| Future | Maintenance mode only | Active development (.NET 8, 9...) |
| Use today | Legacy Windows apps | All new projects |

> **.NET 5** unified the two runtimes. **.NET 8** (LTS) is the current recommended version.

---

### CLR — Common Language Runtime

> The **execution engine** of .NET. Compiles IL → machine code and manages the runtime.

**Compilation flow:**
```
C# Source → C# Compiler → IL (Intermediate Language / .dll) → CLR JIT → Native Machine Code
```

**CLR responsibilities:**

| Responsibility | What it does |
|---|---|
| **JIT Compilation** | Just-In-Time: converts IL to native code at runtime |
| **Garbage Collection** | Automatically frees heap memory (Gen 0/1/2) |
| **Type Safety** | Ensures type rules are enforced at runtime |
| **Exception Handling** | Unified exception model across all .NET languages |
| **Thread Management** | Thread creation, scheduling, synchronization |
| **Security** | Code access security, sandboxing |

**CTS (Common Type System):** Defines how types are declared and used across all .NET languages. `int` in C# = `Integer` in VB.NET = same underlying CTS type.

**CLS (Common Language Specification):** A subset of CTS — rules that ensure cross-language interoperability.

---

### Dependency Injection (DI)

> A pattern where a class **receives its dependencies from outside** rather than creating them itself.

**Without DI (bad):**
- `new EmailService()` inside a controller → tightly coupled, impossible to unit test
- Changing the implementation means changing every caller

**With DI (good):**
- Controller declares `IEmailService` in its constructor
- The **DI container** (IoC container) resolves and injects the right implementation
- Swap `SmtpEmailService` with `SendGridEmailService` in one place — nothing else changes

**Three lifetimes in ASP.NET Core:**

| Lifetime | Created | Destroyed | Use for |
|---|---|---|---|
| **Transient** | Every injection | After use | Stateless lightweight services |
| **Scoped** | Once per HTTP request | End of request | DbContext, per-request work |
| **Singleton** | Once, app startup | App shutdown | Cache, config, logging |

> **Critical interview point:** DbContext must be **Scoped** — never Singleton. A singleton DbContext is shared across all requests, causing change-tracker corruption and race conditions.

> See **Module 5 → Section 5.11** for the full explanation with code.

---

---

# ☀️ DAY 2 — ASP.NET Core + EF Core

> **Goal:** Be confident in building and explaining REST APIs.

---

## 1. Controllers & Routing

### What is a Controller?
> A controller is a C# class that **handles HTTP requests** and returns responses. It contains **action methods** — one per endpoint.

- MVC: inherits `Controller` — can return Views (HTML)
- Web API: inherits `ControllerBase` — returns JSON/XML only
- `[ApiController]` attribute enables automatic validation, binding, and ProblemDetails

### Routing types

| Type | How | Example |
|---|---|---|
| **Convention-based** | Defined once in `Program.cs` | `{controller}/{action}/{id?}` |
| **Attribute routing** | On the controller/action directly | `[Route("api/employees/{id}")]` |

> Web API controllers use **attribute routing exclusively** when `[ApiController]` is applied.

**HTTP verb attributes:** `[HttpGet]`, `[HttpPost]`, `[HttpPut]`, `[HttpPatch]`, `[HttpDelete]`

---

## 2. Dependency Injection (Scoped / Transient / Singleton)

> Already covered in Day 1 above. Key interview additions:

**Registration in `Program.cs`:**
- `builder.Services.AddTransient<IService, Service>()`
- `builder.Services.AddScoped<IService, Service>()`
- `builder.Services.AddSingleton<IService, Service>()`

**Constructor injection** is the standard pattern in ASP.NET Core. The framework reads the constructor parameters and resolves them from the container automatically.

**Captive dependency anti-pattern:** A Singleton service that depends on a Scoped service — the Scoped service is captured in the Singleton and never released per-request. ASP.NET Core throws an `InvalidOperationException` to prevent this.

---

## 3. Middleware Pipeline

> Middleware is a chain of components that process HTTP requests and responses **in sequence**.

**Analogy:** Think of airport security — you pass through check-in → security → gate in that exact order. Each step can stop you or let you through.

**Key principles:**
- Each middleware can run code **before** and **after** calling the next middleware
- **Order matters** — `UseAuthentication` must come before `UseAuthorization`
- `UseExceptionHandler` must be **first** to catch exceptions from anything below it

| Method | Calls next? | Purpose |
|---|---|---|
| `app.Use` | ✅ Yes | Normal middleware — before + after logic |
| `app.Run` | ❌ No (terminal) | Ends the pipeline, writes the final response |
| `app.Map` | Branches | Different pipeline for a path prefix |

> See **Module 5 → Section 5.12** for `app.Use` vs `app.Run` vs `app.Map` with execution order.

---

## 4. Model Binding + Data Annotations

### Model Binding
> ASP.NET Core automatically **maps incoming request data to action method parameters and model objects**.

Sources it reads from (in order):

| Attribute | Data Source |
|---|---|
| `[FromRoute]` | URL path segments (`/employees/{id}`) |
| `[FromQuery]` | Query string (`?page=2&size=10`) |
| `[FromBody]` | Request body (JSON) |
| `[FromHeader]` | HTTP headers |
| `[FromForm]` | HTML form fields |

With `[ApiController]`, complex types are automatically `[FromBody]` — no need to write it explicitly.

### Data Annotations (Validation)
> Attributes on model properties that define **validation rules**.

| Annotation | Rule |
|---|---|
| `[Required]` | Must not be null/empty |
| `[StringLength(100)]` | Max 100 characters |
| `[Range(0, 1000000)]` | Must be between 0 and 1M |
| `[EmailAddress]` | Must be valid email format |
| `[RegularExpression]` | Must match a regex pattern |
| `[Compare("Password")]` | Must match another property |

With `[ApiController]`: validation runs automatically and returns `400 Bad Request` with error details if `ModelState.IsValid` is false.

---

## 5. Filters (ActionFilter, ExceptionFilter)

> Filters run **code at specific points in the request pipeline** — before/after action execution.

**Filter types and their purpose:**

| Filter Type | Runs When | Common Use |
|---|---|---|
| `IAuthorizationFilter` | Before anything else | Custom auth logic |
| `IResourceFilter` | After auth, before model binding | Caching |
| `IActionFilter` | Before/after the action method | Logging, input manipulation |
| `IExceptionFilter` | When an unhandled exception occurs | Centralized error handling |
| `IResultFilter` | Before/after the result is executed | Response manipulation |

**ActionFilter — the most common:**
- `OnActionExecuting` — runs **before** the action
- `OnActionExecuted` — runs **after** the action

> *Use cases:* Logging request/response, validating custom headers, measuring execution time, caching responses.

**ExceptionFilter vs Middleware:**
- `IExceptionFilter` only handles exceptions from **controller actions**
- Middleware (`UseExceptionHandler`) handles exceptions from **anywhere** in the pipeline — preferred for global error handling

---

## 6. Authentication & Authorization

### Authentication vs Authorization

| Concept | Question it answers | Mechanism |
|---|---|---|
| **Authentication** | *Who are you?* — Verify identity | JWT token, Cookie, API Key |
| **Authorization** | *What can you do?* — Check permissions | `[Authorize]`, Roles, Policies |

**ASP.NET Core pipeline order — always:**
```
app.UseAuthentication()  ← reads the token, populates User (ClaimsPrincipal)
app.UseAuthorization()   ← checks [Authorize] attributes
```

### Role-Based Authorization
- `[Authorize(Roles = "Admin")]` — only Admin
- `[Authorize(Roles = "Admin,Manager")]` — Admin OR Manager
- Named policies: `builder.Services.AddAuthorization(options => options.AddPolicy(...))`

> See **Module 6 → Section 6.16** for full RBAC code.

---

## 7. JWT Token Flow

> **JSON Web Token** — a self-contained, stateless token carrying user identity and claims.

**Structure:** `Header.Payload.Signature` — three Base64-encoded parts separated by dots.

**Full flow:**

```
1. User sends username + password to /api/auth/login
2. Server validates credentials against DB
3. Server creates JWT:
   - Header:    algorithm (HS256), type (JWT)
   - Payload:   claims (userId, email, roles, expiry)
   - Signature: HMAC-SHA256(header + payload + secret key)
4. Server returns the JWT to the client
5. Client stores JWT (localStorage / memory)
6. Client sends JWT in every request:
   Authorization: Bearer <token>
7. Server middleware validates signature + expiry on every request
8. If valid → User (ClaimsPrincipal) is populated → action runs
```

**Why stateless?** The server does NOT store the token anywhere. All information is inside the token, verified by the signature. This scales horizontally — any server can validate any token.

**Key claims:** `sub` (subject/userId), `email`, `role`, `exp` (expiry), `iat` (issued at).

> See **Module 6 → Section 6.10** for full JWT implementation.

---

## 8. appsettings.json & IConfiguration

> Configuration values (connection strings, secrets, feature flags) are stored in `appsettings.json` and accessed through `IConfiguration`.

**Environment-specific overrides:**
```
appsettings.json              ← Base (shared by all environments)
appsettings.Development.json  ← Overrides base for Development
appsettings.Production.json   ← Overrides base for Production
```

ASP.NET Core automatically loads the right file based on `ASPNETCORE_ENVIRONMENT`.

**Secrets management:**
- Never put passwords/keys in `appsettings.json` committed to Git
- Use **User Secrets** (dev) → `dotnet user-secrets set "Jwt:Key" "..."
- Use **Azure Key Vault** / environment variables (production)

---

## 9. EF Core — DbContext & DbSet

### DbContext
> The **heart of EF Core** — represents a session with the database. It tracks changes, manages connections, and translates LINQ to SQL.

**Responsibilities:**
- Track entity changes (Added, Modified, Deleted, Unchanged)
- Coordinate `SaveChangesAsync()` — generates the SQL and executes it
- Maintain relationships between entities
- Hold `DbSet<T>` properties (one per table)

### DbSet\<T\>
> Represents a **table in the database**. Querying a `DbSet` builds a SQL query; materializing it (`.ToList()`) sends it.

### Change Tracker
> EF Core keeps an internal snapshot of every entity it loaded. On `SaveChangesAsync()`, it compares current state to snapshot → generates only the necessary `INSERT`/`UPDATE`/`DELETE` SQL.

> **`AsNoTracking()`** — disables the change tracker for a query. Use for **read-only** queries (dashboards, reports) for much better performance.

---

## 10. Code-First Migrations

> Define your **C# model classes first**, then generate the database schema from them.

**Workflow:**
```
1. Create/modify entity class
2. dotnet ef migrations add MigrationName   → generates snapshot diff
3. dotnet ef database update                → applies SQL to DB
```

**What migrations generate:** A C# file with `Up()` (apply) and `Down()` (rollback) methods.

**Why use migrations?**
- Database schema is version-controlled alongside code
- Team members can `database update` to sync locally
- Rollback a bad migration with `migrations remove` or `database update PreviousMigration`

---

## 11. Navigation Properties & Relationships

> Navigation properties are the C# way of expressing **relationships between entities** (FK relationships in the database).

| Relationship | C# Shape | Example |
|---|---|---|
| **One-to-Many** | Parent has `ICollection<Child>`, Child has parent reference | `Department` has many `Employees` |
| **One-to-One** | Both have single reference to each other | `Employee` has one `Address` |
| **Many-to-Many** | Both have `ICollection` of each other | `Student` ↔ `Course` |

**Cascading deletes:** When a parent is deleted, EF Core can automatically delete its children. Configured via `OnDelete(DeleteBehavior.Cascade / Restrict)`.

---

## 12. Eager vs Lazy Loading

| Loading Type | How | When data loads | Risk |
|---|---|---|---|
| **Eager Loading** | `.Include(e => e.Department)` | Same query as parent | None — explicit and predictable |
| **Lazy Loading** | Access navigation property | First time you access it | **N+1 problem** |
| **Explicit Loading** | `context.Entry(emp).Reference(e => e.Dept).LoadAsync()` | When you explicitly call Load | Controlled, but manual |

### N+1 Problem (critical interview topic)
> Loading a list of 100 employees, then accessing `emp.Department` for each → EF Core issues **1 + 100 = 101 queries** instead of 1 JOIN query.

**Fix:** Always use `.Include()` for navigation properties you know you'll need.

---

## 13. LINQ-to-Entities

> When you write LINQ against a `DbSet<T>`, EF Core **translates the LINQ expression tree into SQL** — the filtering, sorting, and joining all happen in the database, not in C#.

**Critical distinction:**

| | Runs where | Loads data |
|---|---|---|
| `IQueryable` (EF Core) | **Database** | Only matching rows |
| `IEnumerable` (after `.ToList()`) | **C# memory** | Already loaded |

> The moment you call `.ToList()`, `.FirstOrDefault()`, `.Count()` etc. — the SQL executes and results come into memory. Any LINQ after that point runs in C#.

> See **Module 4 → Section 4.13** for the full IEnumerable vs IQueryable vs ICollection breakdown.

---

## 📝 Quick Revision Checklist

### Day 1 — Can you answer these?

- [ ] Explain the 4 OOP pillars with a real-world example each
- [ ] When would you choose an interface over an abstract class?
- [ ] What is the difference between a value type and reference type?
- [ ] What is a delegate? How does `Func<T, TResult>` relate to it?
- [ ] Why should you never use `.Result` or `.Wait()` in async code?
- [ ] What is deferred execution in LINQ?
- [ ] What does the CLR do?
- [ ] What are the three DI lifetimes? Why is DbContext Scoped?

### Day 2 — Can you answer these?

- [ ] What is the difference between authentication and authorization?
- [ ] What does the middleware pipeline look like in `Program.cs`? Why does order matter?
- [ ] What does `[ApiController]` do? Name all four behaviors.
- [ ] Explain the JWT token flow end to end
- [ ] What is the N+1 problem? How do you fix it?
- [ ] What is the difference between `IQueryable` and `IEnumerable` in EF Core?
- [ ] What is a migration? What does `Up()` and `Down()` do?
- [ ] What is `AsNoTracking()` and when do you use it?
- [ ] What is an `IActionFilter`? Give a use case.

---

## 🔗 Cross-References

| Topic | Detailed File |
|---|---|
| CLR, async/await, Extension Methods, Reflection | [Module 1](./Module1_DotNET8_CSharp12.md) |
| SOLID Principles | [Module 2](./Module2_SOLID_Principles.md) |
| EF Core — DbContext, LINQ, Fluent API, IQueryable | [Module 4](./Module4_EFCore_Dapper.md) |
| Middleware, DI lifetimes, Routing, Filters | [Module 5](./Module5_ASPNETCore8.md) |
| JWT, [ApiController], RBAC, MVC vs Web API | [Module 6](./Module6_WebAPI.md) |

---

_[← Back to README](./README.md)_
