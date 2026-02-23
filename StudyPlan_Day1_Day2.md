# 📅 Study Plan — Day 1 & Day 2 (Detailed Theory)

> **Theory-first, interview-ready.** Each topic has: What it is → Why it matters → How it works → Interview Answer.

---

# ☀️ DAY 1 — C# + OOP + .NET Basics

---

## 1. OOP — The Four Pillars

### 🔷 Encapsulation

**What is it?**
Encapsulation means **bundling data (fields) and the methods that operate on that data together in one class**, and **restricting direct access to the internal state** from outside the class.

**Why it matters:**
Without encapsulation, any code anywhere can change an object's internal data to an invalid state. For example, setting `account.Balance = -999999` directly. Encapsulation forces all changes to go through controlled methods that can validate the input.

**How it works:**
- Mark fields `private` — they cannot be accessed from outside
- Expose only what is needed through `public` properties or methods
- Properties can have logic in `get`/`set` — e.g., validate before setting

**Access modifiers in C#:**
| Modifier | Accessible from |
|---|---|
| `private` | Only within the same class |
| `protected` | Same class + subclasses |
| `internal` | Same assembly (project) |
| `protected internal` | Same assembly OR subclasses |
| `public` | Anywhere |

**Real example:**
A `BankAccount` class has a `private decimal _balance` field. The only way to change it is through `Deposit(amount)` and `Withdraw(amount)` methods — which validate that amount > 0 and balance doesn't go negative. No external code can bypass this logic.

**Interview Answer:** *"Encapsulation is about hiding the internal state of an object and only exposing behaviour through a defined interface. We use private fields and public properties/methods. This prevents objects from being put into invalid states and makes the code more maintainable — if the internal implementation changes, callers don't need to change as long as the public interface stays the same."*

---

### 🔷 Abstraction

**What is it?**
Abstraction means **showing only the essential features** of an object and hiding the implementation details. The user of a class knows *what* it does, not *how* it does it.

**Why it matters:**
It reduces complexity. A developer calling `emailService.SendAsync(to, subject, body)` doesn't need to understand SMTP handshakes, retry logic, or connection pooling. They just use the clean interface.

**How it works:**
- **Interfaces** — pure contracts, no implementation
- **Abstract classes** — partial abstraction, can have some implemented methods
- Access modifiers also contribute — `private` methods are implementation details hidden from callers

**The difference between Abstraction and Encapsulation:**
- **Encapsulation** = *hiding data* (fields) — about protecting state
- **Abstraction** = *hiding complexity* (implementation) — about simplifying the interface

**Real example:**
EF Core's `DbContext.SaveChangesAsync()` — you call one method and it internally: detects all changed entities via the change tracker, generates parameterized SQL for each change (INSERT/UPDATE/DELETE), wraps them in a transaction, executes them in the right order, handles concurrency conflicts. You see none of that. That's abstraction.

**Interview Answer:** *"Abstraction means exposing only what is needed and hiding the implementation complexity. It's achieved through interfaces and abstract classes. The caller works with a contract — IEmailService, IRepository — without knowing whether emails are sent via SMTP or SendGrid, or whether data comes from SQL or a cache. This makes code easier to use, test, and change."*

---

### 🔷 Inheritance

**What is it?**
Inheritance allows a class (child/derived) to **acquire the properties and behaviour of another class** (parent/base). The child class gets all non-private members of the parent and can extend or override them.

**Why it matters:**
Eliminates code duplication. Common behaviour defined in one place — all subclasses benefit automatically. If you fix a bug in the parent, all children are fixed.

**How it works in C#:**
- Syntax: `class Dog : Animal` — Dog inherits from Animal
- `virtual` keyword on parent method = can be overridden
- `override` keyword on child method = overrides the parent's version
- `base.Method()` = call the parent's version from the child
- `sealed` on a class = no further inheritance allowed
- C# supports **single inheritance** only — one base class per class
- Multiple inheritance of behaviour → use interfaces

**Types of relationships:**
| Keyword | Meaning |
|---|---|
| `:` (no keyword) | Inherits / Implements |
| `virtual` | Parent method can be overridden |
| `override` | Child overrides parent's virtual method |
| `abstract` | Parent has no implementation — child MUST override |
| `sealed` | Cannot be further inherited |
| `base` | Access parent's members from child |
| `new` | Hides parent's method (not polymorphic — avoid) |

**Real example:**
`EmployeeController : ControllerBase` — your controller inherits `Ok()`, `NotFound()`, `BadRequest()`, `CreatedAtAction()` and many more response helper methods from `ControllerBase`. You don't write any of that code — it comes from the parent.

**Interview Answer:** *"Inheritance lets a derived class reuse code from a base class. The child gets all non-private members of the parent. In C# we only have single inheritance for classes, so a class can extend only one base. We use `virtual`/`override` for runtime polymorphism and `abstract` to force subclasses to provide their own implementation."*

---

### 🔷 Polymorphism

**What is it?**
Polymorphism means **"many forms"** — the same method call behaves differently depending on the actual type of the object at runtime.

**Why it matters:**
It's what allows you to write `foreach (var shape in shapes) { shape.Draw(); }` without knowing or caring whether each shape is a Circle, Square, or Triangle. The correct `Draw()` method is called automatically. This is the foundation of extensibility — add a new shape by creating a new class; the calling code never changes.

**Two types:**

| Type | Mechanism | Resolved when |
|---|---|---|
| **Compile-time (Static)** | Method **overloading** — same method name, different parameter signatures | At compile time |
| **Runtime (Dynamic)** | Method **overriding** — `virtual` in base, `override` in derived | At runtime (based on actual object type) |

**Method Overloading vs Overriding:**
- **Overloading**: Same class, same method name, different parameters. `Calculate(int x)` and `Calculate(int x, int y)` are different methods — the compiler picks the right one based on the arguments.
- **Overriding**: Subclass replaces parent's method behaviour. The runtime decides which version to call based on the actual object type, not the variable type.

**How virtual dispatch works:**
When you call `shape.Draw()` and `shape` is declared as `Shape` but the actual object is a `Circle`, the CLR uses a **vtable (virtual method table)** to look up the correct `Circle.Draw()` method at runtime. This is called **late binding**.

**Real example:**
```
Shape[] shapes = { new Circle(), new Square(), new Triangle() };
foreach (var s in shapes)
    s.Draw();  // Circle draws itself, Square draws itself, Triangle draws itself
               // All through one unified call — polymorphism
```

**Interview Answer:** *"Polymorphism means the same method call can behave differently based on the object's actual type. Compile-time polymorphism is method overloading — resolved by the compiler based on parameter types. Runtime polymorphism is method overriding with `virtual`/`override` — the CLR uses a vtable to call the correct derived class method at runtime. This is fundamental to the Open/Closed principle — you can add new types without changing existing code."*

---

## 2. Interfaces vs Abstract Classes

**What is an Interface?**
An interface is a **pure contract** — it defines what methods and properties a class must have, but provides no state and (before C# 8) no implementation. A class that implements an interface promises to provide all the listed members.

**What is an Abstract Class?**
An abstract class is a **partial implementation** — it can have both abstract members (no body — must be overridden) and concrete members (have a body — shared by all subclasses). It cannot be instantiated directly.

**Deep comparison:**

| Aspect | Interface | Abstract Class |
|---|---|---|
| Instantiate directly | ❌ No | ❌ No |
| Fields / instance variables | ❌ No | ✅ Yes |
| Constructor | ❌ No | ✅ Yes |
| Concrete (implemented) methods | ✅ Default methods only (C# 8+) | ✅ Yes, fully |
| Multiple "inheritance" | ✅ A class can implement many interfaces | ❌ Only one base class |
| Access modifiers on members | All public (implicitly) | Any (private, protected, public) |
| Relationship expressed | "**can-do**" capability | "**is-a**" relationship |
| Versioning | Adding a member breaks all implementors | Can add concrete methods without breaking subclasses |

**When to use Interface:**
- When **unrelated classes** need to share a contract
- When you want to support **multiple inheritance** of behaviour
- When you want to enable **unit testing** via mocking (you mock `IEmailService` in tests)
- Examples: `ILogger`, `IRepository<T>`, `IEmailService`, `IPaymentGateway`

**When to use Abstract Class:**
- When classes share **common state** (fields) or common **implementation**
- When there is a genuine **is-a** relationship with shared logic
- When you want to provide a **template method** — define the algorithm skeleton, let subclasses fill in specific steps
- Examples: `Animal` (has `Breathe()` concrete, `Sound()` abstract), `BaseController`, `RepositoryBase<T>`

**The practical rule:**
> If you find yourself repeating logic across multiple classes that all implement the same interface, move that common logic into an abstract base class that implements the interface.

**Interview Answer:** *"Interfaces define a contract — what methods a class must have — with no state or implementation. Abstract classes can have both abstract (must override) and concrete (shared) methods, plus state. A class can implement multiple interfaces but only inherit one abstract class. I use interfaces to decouple components and enable mocking in unit tests. I use abstract classes when there is genuinely shared logic or state across subclasses."*

---

## 3. Value Types vs Reference Types

**What are Value Types?**
Value types hold their data **directly in the variable**. They are stored on the **stack** (when local variables) or inline within a containing object. When you assign one value type to another, a **copy** is made.

**What are Reference Types?**
Reference types store a **pointer (reference) to the actual data** on the **heap**. When you assign one reference type variable to another, both variables point to the **same object on the heap**.

**Deep comparison:**

| Aspect | Value Type | Reference Type |
|---|---|---|
| Stored | Stack (or inline in object on heap) | Heap (variable holds a reference) |
| Assignment | **Copies the value** | **Copies the reference** (not the object) |
| Equality (default) | By **value** | By **reference** (same object?) |
| Null | ❌ Cannot be null (use `int?` for nullable) | ✅ Can be null |
| Default value | `0`, `false`, `\0` | `null` |
| Examples | `int`, `double`, `bool`, `char`, `decimal`, `struct`, `enum` | `class`, `string`, `array`, `interface`, `delegate` |
| GC involvement | ❌ No (stack memory auto-freed) | ✅ Yes (GC manages heap) |

**The mutation trap:**
```
// Reference type — both variables point to same object
Employee emp1 = new Employee { Name = "Alice" };
Employee emp2 = emp1;             // emp2 is NOT a copy — same object!
emp2.Name = "Bob";
Console.WriteLine(emp1.Name);     // Prints "Bob" ← emp1 is also changed!

// Value type — copy is made
int a = 5;
int b = a;                        // b is a separate copy
b = 10;
Console.WriteLine(a);             // Prints 5 ← a is unchanged
```

**String is special:**
`string` is a **reference type** but behaves like a value type because strings are **immutable**. Every operation that "changes" a string creates a new string object on the heap. This is why `string s = s.ToUpper()` works correctly — `s` now points to a new string.

**Boxing and Unboxing:**
When a value type is assigned to an `object` variable, it gets **boxed** — wrapped in a heap object. Converting back is **unboxing**. This incurs a performance cost and should be avoided in performance-sensitive code.

**Interview Answer:** *"Value types (int, struct, enum) store data directly and are copied on assignment. Reference types (class, arrays) store a pointer to heap data — assignment copies the reference, so two variables can point to the same object. This is why modifying an Employee via one variable changes what the other variable sees. Strings are reference types but immutable, so they behave like value types. Boxing occurs when a value type is converted to object, which has a heap allocation cost."*

---

## 4. Delegates, Func, Action

**What is a Delegate?**
A delegate is a **type-safe function pointer** — a type that holds a reference to a method with a specific signature. You can pass methods as parameters, store them in variables, and call them through the delegate.

**Why delegates exist:**
Before delegates, you couldn't pass a method as a parameter — you needed interfaces with a single method (like `IComparable`). Delegates are more lightweight and direct.

**Built-in delegate types (no need to define your own):**

| Type | Returns | Use case |
|---|---|---|
| `Action` | `void` | Do something, return nothing |
| `Action<T>` | `void` | Do something with input T |
| `Action<T1, T2>` | `void` | Do something with two inputs |
| `Func<TResult>` | `TResult` | Return a value, no input |
| `Func<T, TResult>` | `TResult` | Take T, return TResult |
| `Func<T1, T2, TResult>` | `TResult` | Take two inputs, return TResult |
| `Predicate<T>` | `bool` | Test a condition — shorthand for `Func<T, bool>` |

**Where you encounter delegates in real code:**
- `List<T>.ForEach(Action<T> action)` — `Action` delegate
- `LINQ .Where(Func<T, bool> predicate)` — `Func` delegate
- `LINQ .Select(Func<T, TResult> selector)` — `Func` delegate
- Event handlers: `button.Click += OnButtonClicked;` — `EventHandler` delegate
- Middleware registration: `app.Use(async (context, next) => { ... })` — delegate

**Multicast Delegates:**
A delegate can hold references to **multiple methods**. When invoked, all methods are called in order. This is the basis of C# events (`+=` adds a handler, `-=` removes one).

**Interview Answer:** *"A delegate is a type-safe function pointer — it holds a reference to a method with a matching signature. `Action<T>` is a built-in delegate that takes a parameter and returns void. `Func<T, TResult>` takes T and returns TResult. These are used everywhere in LINQ (Where, Select all take Func delegates), event handling, and passing behaviour as a parameter. Lambda expressions are just syntactic sugar for creating anonymous delegates inline."*

---

## 5. Lambda Expressions

**What is a Lambda?**
A lambda is an **anonymous (unnamed) function defined inline** — a shorthand for writing a delegate without declaring a separate named method.

**Syntax:** `(parameters) => expression` or `(parameters) => { statements; }`

**Why lambdas exist:**
Without lambdas you'd have to write a named method just to use it once in a LINQ query. Lambdas make intent clearer and reduce boilerplate.

**How lambdas work internally:**
The compiler converts a lambda expression into either:
1. An **anonymous method** (delegate) — when used with `Func`/`Action`
2. An **expression tree** (`Expression<Func<T, bool>>`) — when used with `IQueryable`. EF Core reads the expression tree and **translates it into SQL**. This is the core of LINQ-to-Entities.

**Lambda as expression vs statement:**
- Expression lambda: `e => e.Salary > 50000` — single expression, no braces
- Statement lambda: `e => { var x = e.Salary; return x > 50000; }` — multiple statements, needs braces and return

**Closures:**
Lambdas can capture variables from their enclosing scope — this is called a **closure**. The lambda "closes over" the variable and holds a reference to it, even after the outer method returns.

**Interview Answer:** *"A lambda expression is a concise way to write an anonymous function inline. The compiler translates it into a delegate (when used with `Action`/`Func`) or an expression tree (when used with `IQueryable`). EF Core uses the expression tree to translate LINQ queries into SQL — that's why you can write `Where(e => e.Salary > 50000)` and EF generates `WHERE Salary > 50000` in SQL. The expression tree lets EF inspect the lambda's structure rather than just executing it."*

---

## 6. async / await

**What is it?**
`async`/`await` is C#'s model for writing **asynchronous, non-blocking code** that looks like synchronous code. It is built on top of the **Task Parallel Library (TPL)** and the `Task` type.

**The core problem it solves:**
Traditional synchronous code: when your method calls the database, the current **thread blocks and waits**. Blocked threads consume memory and can't serve other requests. A typical web server has 100–200 threads. With synchronous code you can serve at most ~100–200 concurrent requests.

With async: when `await` is hit, the thread is **released back to the thread pool**. When the I/O completes, any available thread resumes execution. Now your 100 threads can serve thousands of concurrent requests.

**How it works step by step:**
1. Method is marked `async` — compiler rewrites it as a **state machine**
2. When `await someTask` is reached:
   - If the task is already complete → execution continues synchronously (no overhead)
   - If the task is not complete → the current thread is released, execution returns to the caller
3. When the awaited task completes → the runtime picks up a thread from the pool and resumes the state machine from where it left off
4. The final result of the async method is returned as `Task<T>`

**Key rules table:**

| Rule | Explanation |
|---|---|
| Return `Task` or `Task<T>` | `Task` = no return value, `Task<T>` = returns T |
| Mark method `async` | Compiler can use `await` inside and creates state machine |
| Never `.Result` or `.Wait()` | These block the thread synchronously and can **deadlock** in ASP.NET Core — the request context holds a lock that `.Result` tries to re-acquire |
| `async void` only for events | No way to `await` it, exceptions are swallowed — never use in APIs |
| Name with `Async` suffix | Convention — `GetEmployeeAsync`, `SaveChangesAsync` |
| Parallel I/O | `await Task.WhenAll(task1, task2)` — both run simultaneously |
| Sequential I/O | `await task1; await task2;` — runs one after the other |

**Why `.Result` causes deadlock in ASP.NET Core (classic):**
ASP.NET (old) had a synchronization context. When you call `.Result`, the current thread blocks waiting for the task to complete. The task tries to resume on the same synchronization context thread — but that thread is blocked waiting. Deadlock. In ASP.NET Core there is no synchronization context, so deadlocks are less common but `.Result` still wastes threads.

**Interview Answer:** *"async/await enables non-blocking I/O. When `await` is reached, the thread is released back to the thread pool while the I/O operation completes. This is why async Web APIs can handle thousands of concurrent requests with a small thread pool. You should never use `.Result` or `.Wait()` as they block the thread and can cause deadlocks. The compiler rewrites async methods as state machines under the hood."*

---

## 7. LINQ — Language Integrated Query

**What is it?**
LINQ is a set of **extension methods on `IEnumerable<T>` and `IQueryable<T>`** that let you query, filter, transform, and aggregate collections using C# syntax — consistently, regardless of the data source (in-memory lists, databases, XML, etc.).

**Two execution models:**

| Mode | Interface | Where it runs | Used with |
|---|---|---|---|
| **LINQ to Objects** | `IEnumerable<T>` | In C# memory | Lists, arrays, in-memory collections |
| **LINQ to Entities** | `IQueryable<T>` | In the database (SQL) | EF Core DbSet |

**Deferred execution — critical concept:**
LINQ queries are **not executed when you define them**. They build up an expression. Execution happens when:
- `.ToList()` or `.ToArray()` is called
- `.Count()`, `.First()`, `.Any()` is called
- You iterate with `foreach`

This means you can build a query in parts, add conditions conditionally, and then execute it once — EF Core sends one efficient SQL query.

**Key LINQ operators with explanations:**

| Operator | What it does | SQL equivalent |
|---|---|---|
| `Where(predicate)` | Filter rows | `WHERE` |
| `Select(selector)` | Transform/project each row | `SELECT col1, col2` |
| `OrderBy` / `OrderByDescending` | Sort | `ORDER BY ... ASC/DESC` |
| `GroupBy(keySelector)` | Group rows by a key | `GROUP BY` |
| `Join(inner, outerKey, innerKey, result)` | Inner join two collections | `INNER JOIN` |
| `FirstOrDefault(predicate)` | First match or null | `SELECT TOP 1 ... WHERE` |
| `SingleOrDefault(predicate)` | Exactly one match or null (throws if >1) | — |
| `Any(predicate)` | Does any row match? Returns bool | `EXISTS (SELECT ...)` |
| `All(predicate)` | Do all rows match? Returns bool | — |
| `Count(predicate)` | Count rows | `COUNT(*)` |
| `Sum` / `Average` / `Max` / `Min` | Aggregate | `SUM` / `AVG` / `MAX` / `MIN` |
| `Include(nav)` | EF Core: JOIN navigation property | `JOIN` in generated SQL |
| `Skip(n).Take(m)` | Pagination | `OFFSET n ROWS FETCH NEXT m ROWS ONLY` |
| `Distinct()` | Remove duplicates | `SELECT DISTINCT` |
| `SelectMany` | Flatten nested collections | Multiple rows per parent |

**Method syntax vs Query syntax:**
LINQ has two syntaxes — both compile to the same thing:
```
// Method syntax (most common in practice)
var result = employees.Where(e => e.Dept == "IT").Select(e => e.Name);

// Query syntax (SQL-like)
var result = from e in employees where e.Dept == "IT" select e.Name;
```

**Interview Answer:** *"LINQ provides a unified query syntax over any data source. With `IEnumerable`, LINQ runs in C# memory. With `IQueryable` (EF Core), LINQ builds an expression tree that EF translates into SQL — so filtering and sorting happen in the database, not in memory. Queries are lazily evaluated — they don't run until you call ToList() or similar. This allows composing queries conditionally before executing once."*

---

## 8. Exception Handling

**What is it?**
Exception handling is the structured mechanism for dealing with **runtime errors** — using `try`, `catch`, `finally`, and `throw` to handle and propagate errors in a controlled way.

**How the exception mechanism works:**
1. An exception is thrown (by runtime or `throw new ...`)
2. The CLR walks up the **call stack** looking for a matching `catch` block
3. The first matching `catch` handles it
4. `finally` always runs — whether or not an exception occurred
5. If no handler is found → unhandled exception → process terminates

**Key keywords:**

| Keyword | Purpose | Important detail |
|---|---|---|
| `try` | Wrap code that might fail | — |
| `catch (SpecificException ex)` | Handle a specific type | Most specific first — catches subclasses too |
| `catch (Exception ex)` | Catch-all | Use last; avoid swallowing exceptions silently |
| `finally` | Always runs after try/catch | Use for cleanup: close connections, dispose objects |
| `throw` | Re-throw current exception | **Preserves** the original stack trace |
| `throw ex` | Re-throw a caught exception | **Resets** stack trace — hides the origin — avoid! |
| `throw new Ex(msg, innerEx)` | Wrap and re-throw | Preserves original as `InnerException` |
| `when (condition)` | Exception filter | Catches only if condition is true |

**Custom exceptions:**
Inherit from `Exception` to create meaningful domain exceptions:
- `NotFoundException` — when a resource doesn't exist (maps to 404)
- `ValidationException` — when input is invalid (maps to 400)
- `UnauthorizedException` — when access is denied (maps to 401/403)

**Exception handling layers in ASP.NET Core Web API:**
1. **Action Filter (`IExceptionFilter`)** — handles exceptions from controller actions only
2. **Middleware (`UseExceptionHandler`)** — global, catches everything in the pipeline, converts to `ProblemDetails` response
3. **`ProblemDetails` (RFC 7807)** — standardized JSON error response format: `{ "type", "title", "status", "detail", "traceId" }`

**What NOT to do:**
- Empty `catch` blocks — silently swallows errors
- Catching `Exception` at every level — handle at the right layer
- Using exceptions for flow control — exceptions are expensive, don't use them for expected scenarios like "user not found"

**Interview Answer:** *"Exception handling uses try/catch/finally. Catch the most specific exceptions first — they're checked top to bottom and the first match wins. Use `throw` (not `throw ex`) to re-throw, preserving the stack trace. `finally` runs in all cases and is used for cleanup. In ASP.NET Core Web APIs, global exception handling is done with middleware (`UseExceptionHandler`) which catches all unhandled exceptions and converts them to a standardised ProblemDetails response."*

---

## 9. SOLID Principles

### S — Single Responsibility Principle (SRP)
**Statement:** A class should have **one, and only one, reason to change**.

**What it really means:** Every class should have one clearly defined job. If a class handles business logic, database access, AND email sending — it has three reasons to change (business rule change, DB schema change, email provider change). That's a violation.

**Violation symptom:** A class that has the words "Manager", "Helper", "Processor" or is 500+ lines long and does many unrelated things.

**Fix:** Split into separate classes: `OrderService` (business logic), `OrderRepository` (data access), `OrderNotificationService` (emails).

---

### O — Open/Closed Principle (OCP)
**Statement:** Software entities should be **open for extension, closed for modification**.

**What it really means:** When requirements change, you should be able to add new behaviour by **adding new code**, not by editing existing, tested code. Editing existing code risks breaking what already works.

**How to achieve it:** Use abstractions (interfaces/abstract classes). New implementations extend the abstraction without changing the consumers.

**Violation symptom:** A method with a large `if/else` or `switch` that checks a type and you must add a new case every time a new type is added. Every addition modifies the existing method.

**Fix:** Define an interface (`IDiscount`), implement a new class for each type (`StudentDiscount`, `SeniorDiscount`), register the right one via DI. The consumer never changes.

---

### L — Liskov Substitution Principle (LSP)
**Statement:** If S is a subtype of T, then objects of type T may be replaced with objects of type S **without altering the correctness of the program**.

**What it really means:** A derived class must be a proper substitute for its base class. You should be able to use a `Dog` anywhere an `Animal` is expected, and things should work correctly. If the derived class breaks the contract of the base class — LSP is violated.

**Classic violation:** `Square` inherits from `Rectangle`. `Rectangle.SetWidth(5)` and `Rectangle.SetHeight(3)` should give `Area() = 15`. But if `Square` overrides both setters to always keep Width == Height, then `Area()` is no longer 15 — it's 9. The `Square` is NOT a valid substitute for `Rectangle`.

**Violation symptom:** Throwing `NotImplementedException` in an overridden method, or overriding a method to do nothing, or adding special `if (this is SpecificType)` checks in callers.

**Fix:** Redesign the hierarchy so derived classes truly extend without weakening the base contract.

---

### I — Interface Segregation Principle (ISP)
**Statement:** Clients should not be forced to depend on interfaces they do not use.

**What it really means:** Don't create one fat interface with 20 methods. Split it into small, focused interfaces. A class that implements the interface only needs to implement what it actually uses.

**Violation symptom:** An interface `IAnimal` has `Walk()`, `Fly()`, `Swim()`. A `Dog` class is forced to implement `Fly()` (which it can't do) and throws `NotImplementedException`.

**Fix:** Split into `IWalkable`, `IFlyable`, `ISwimmable`. Dog implements `IWalkable` and `ISwimmable`. Bird implements `IWalkable` and `IFlyable`.

**In practice (ASP.NET Core):** `IRepository<T>` with just `GetById`, `GetAll`, `Add`, `Update`, `Delete` is a well-segregated interface. Don't put `SendEmail()` into it.

---

### D — Dependency Inversion Principle (DIP)
**Statement:** High-level modules should not depend on low-level modules. **Both should depend on abstractions.**

**What it really means:** Your business logic (`OrderService`) should not directly instantiate and use a concrete `SqlOrderRepository`. If you change from SQL to MongoDB, `OrderService` breaks. Instead, `OrderService` should depend on `IOrderRepository` — and the DI container injects the right implementation.

**How DI in ASP.NET Core implements DIP:**
- You register `IOrderRepository → SqlOrderRepository` in `Program.cs`
- `OrderService` declares `IOrderRepository` in its constructor — it never knows the concrete type
- To switch to MongoDB: create `MongoOrderRepository`, change one line in `Program.cs`. `OrderService` doesn't change.

**This is why DI and SOLID go hand in hand.** DI containers are the mechanism that makes DIP practical.

---

## 10. .NET Core vs .NET Framework

**The history:**
- **.NET Framework** (2002–) — Windows-only, closed source, shipped with Windows, mature but slow to evolve
- **.NET Core** (2016–) — cross-platform, open source, high performance, modular
- **.NET 5** (2020) — unified both into a single platform called just ".NET"
- **.NET 8** (2023, LTS) — current recommended version

**Key differences:**

| Aspect | .NET Framework | .NET Core / .NET 5+ |
|---|---|---|
| Platform | Windows only | Windows, Linux, macOS |
| Open source | ❌ | ✅ (GitHub: dotnet/runtime) |
| Performance | Moderate | Significantly faster (esp. Kestrel web server) |
| Deployment | Framework must be installed on machine | **Self-contained** — ship the runtime with your app |
| Side-by-side | ❌ One version per machine | ✅ Multiple versions side-by-side |
| WinForms / WPF | ✅ | ✅ (.NET 6+ supports Windows UI) |
| ASP.NET Web Forms | ✅ | ❌ Not ported |
| Future | Maintenance mode — no new features | ✅ Active development |

**Why .NET Core is faster:**
- Built from scratch with performance as a goal
- Kestrel web server is among the fastest HTTP servers in benchmarks
- Span\<T\>, Memory\<T\> — zero-allocation memory slices
- Native AOT in .NET 7/8 — compile directly to machine code at build time (no JIT at startup)

---

## 11. CLR, CTS, JIT — In Depth

### CLR — Common Language Runtime

**What is it?**
The CLR is the **virtual machine / execution engine** of .NET. It takes compiled code (IL) and makes it run on the actual hardware. It also provides all the runtime services.

**The compilation pipeline:**
```
Step 1: You write C# code                           → YourApp.cs
Step 2: C# Compiler (csc.exe / Roslyn)              → YourApp.dll (contains IL + metadata)
Step 3: At runtime, CLR's JIT Compiler reads IL     → Native machine code (x86/x64)
Step 4: Native code executes on the CPU
```

**Why compile to IL first (not directly to machine code)?**
IL is **platform-neutral**. The same `.dll` runs on Windows x64, Linux ARM, or macOS — the CLR on each platform compiles IL to the appropriate native code. This is how "write once, run anywhere" works in .NET.

**CLR Services — each explained:**

| Service | Detail |
|---|---|
| **JIT Compilation** | Converts IL to native code **method-by-method**, only when each method is first called. Subsequent calls use the cached native code. |
| **Garbage Collection** | Automatically manages heap memory. Objects are allocated on the heap; GC identifies unreachable objects and frees their memory. Uses generational model: Gen 0 (short-lived), Gen 1, Gen 2 (long-lived). |
| **Type Safety** | Verifies IL before execution — ensures no illegal memory access, no invalid casts, no buffer overflows at the IL level. |
| **Exception Handling** | Provides a unified structured exception model across all .NET languages — `try/catch/finally` works the same in C#, VB.NET, F#. |
| **Thread Management** | Creates and manages threads. Provides the Thread Pool — a pool of reusable threads for async work. |
| **Security** | Code Access Security (CAS) for partial-trust environments. Largely replaced by OS-level security in modern .NET. |

**Garbage Collection — Generations:**
- **Gen 0**: Newly created objects. Collected very frequently. Most objects die young (short-lived temps).
- **Gen 1**: Objects that survived Gen 0 collection. Buffer between Gen 0 and Gen 2.
- **Gen 2**: Long-lived objects (static data, caches). Collected infrequently.
- **Large Object Heap (LOH)**: Objects > 85KB — collected with Gen 2, never compacted (historically).

### CTS — Common Type System
**What is it?** CTS defines the **rules and types** that all .NET languages must follow. It ensures that a `System.Int32` (which C# calls `int`, VB.NET calls `Integer`) is the **same binary type** regardless of language.

**Why it matters:** It enables cross-language interoperability. A C# class library can be used from VB.NET or F# without any conversion.

### CLS — Common Language Specification
A **subset of CTS** that defines the minimum set of features that .NET languages must support to be interoperable. If your public API only uses CLS-compliant types, any .NET language can consume it.

---

## 12. Dependency Injection — In Depth

**What is DI?**
Dependency Injection is a **design pattern** where a class **does not create its own dependencies** — it receives them from outside (from a container or caller). This makes classes:
- **Testable** — inject mock dependencies in unit tests
- **Loosely coupled** — depend on abstractions, not concrete classes
- **Flexible** — swap implementations in one place

**The problem without DI:**
```
class OrderService {
    private SqlOrderRepository _repo = new SqlOrderRepository(); // Tightly coupled!
    // Can't test without hitting real DB
    // Can't swap to a different repo without editing this class
}
```

**With DI:**
```
class OrderService {
    private readonly IOrderRepository _repo;
    public OrderService(IOrderRepository repo) { _repo = repo; } // Injected!
    // Test: inject MockOrderRepository
    // Production: inject SqlOrderRepository
    // Tomorrow: inject MongoOrderRepository — this class doesn't change
}
```

**IoC Container (Inversion of Control Container):**
ASP.NET Core has a **built-in IoC container** (`IServiceCollection`). You register services in `Program.cs`, and the container:
1. Knows what type to create when a constructor parameter needs `IOrderRepository`
2. Creates the instance with the correct lifetime
3. Injects it automatically — you never call `new` for your services

**Three lifetimes — when to use each:**

| Lifetime | Instance created | Instance destroyed | When to use |
|---|---|---|---|
| **Transient** | Every time it's requested from container | Immediately after use | Lightweight, stateless services — e.g., email formatter, validators |
| **Scoped** | Once per HTTP request (once per DI scope) | End of HTTP request | Services that need to be consistent within one request — DbContext, repositories, business services |
| **Singleton** | Once when first requested (or app start) | When app shuts down | Shared, thread-safe state — caches, configuration, HTTP clients |

**Why DbContext must be Scoped:**
DbContext has a **change tracker** — it tracks which entities have been loaded and what changes have been made. If DbContext were Singleton, it would be shared across all concurrent requests. User A's changes would be visible to User B before they're saved. The change tracker would get into inconsistent states with multiple threads modifying it simultaneously. Scoped means each HTTP request gets its own fresh DbContext — complete isolation.

**Captive Dependency (Anti-Pattern):**
A **Singleton** service that takes a **Scoped** service in its constructor. The Scoped service gets "captured" inside the Singleton and never changes for the lifetime of the app — it doesn't get a fresh instance per request. ASP.NET Core detects this and throws `InvalidOperationException` at startup in development mode.

**Interview Answer:** *"DI is a pattern where classes receive their dependencies from outside rather than creating them. ASP.NET Core has a built-in IoC container. You register services with one of three lifetimes: Transient (new every time), Scoped (once per HTTP request — use for DbContext), Singleton (once for the whole app — use for caches). DI makes code testable because you can inject mock implementations. It implements the Dependency Inversion Principle — high-level classes depend on abstractions, not concrete implementations."*

---

---

# ☀️ DAY 2 — ASP.NET Core + EF Core

---

## 1. Controllers & Routing

**What is a Controller?**
A controller is a C# class that **receives an HTTP request, processes it, and returns an HTTP response**. In ASP.NET Core Web API, each public method on a controller is an **action** that maps to an endpoint.

**Two base classes:**

| Base Class | Namespace | Use for | Has View support |
|---|---|---|---|
| `Controller` | Microsoft.AspNetCore.Mvc | MVC (returns HTML views) | ✅ Yes |
| `ControllerBase` | Microsoft.AspNetCore.Mvc | Web API (returns JSON/XML) | ❌ No |

> Use `ControllerBase` for APIs — `Controller` adds about 400KB of view-related code you don't need.

**What `[ApiController]` adds (4 behaviours):**
1. **Automatic model validation** — returns `400 Bad Request` automatically if `ModelState.IsValid` is false
2. **Automatic `[FromBody]` inference** — complex type parameters are automatically treated as body
3. **RFC 7807 ProblemDetails** — standardised error response JSON format
4. **Requires attribute routing** — convention routing (`{controller}/{action}`) doesn't work

**Routing types:**

| Type | Where defined | Best for |
|---|---|---|
| **Convention-based** | `Program.cs` in `MapControllerRoute` | MVC apps with consistent URL pattern |
| **Attribute routing** | `[Route]`, `[HttpGet]` on controller/action | Web APIs — explicit, clear, flexible |

**Route tokens and constraints:**
- `[Route("api/[controller]")]` — `[controller]` is replaced with the controller class name (minus "Controller")
- `[HttpGet("{id:int}")]` — `:int` is a route constraint — only matches if id is an integer
- `[HttpGet("{id:int:min(1)}")]` — must be an integer greater than or equal to 1

**Action result types:**

| Return Type | When to use |
|---|---|
| `IActionResult` | Flexible — can return different response types |
| `ActionResult<T>` | Preferred for APIs — strongly typed + Swagger understands it |
| `T` directly | Simplest — always 200 OK with body |

**HTTP Status codes for REST:**
| Action | Success Code | Notes |
|---|---|---|
| GET | `200 OK` | Return the resource |
| POST (create) | `201 Created` | Include `Location` header + `CreatedAtAction` |
| PUT / PATCH | `200 OK` or `204 No Content` | 204 if nothing to return |
| DELETE | `204 No Content` | Nothing to return |
| Validation error | `400 Bad Request` | Model validation failed |
| Not found | `404 Not Found` | Resource doesn't exist |
| Server error | `500 Internal Server Error` | Unhandled exception |

---

## 2. Dependency Injection (Scoped / Transient / Singleton)

> Covered in depth in Day 1 → Section 12. Key additions for Day 2:

**Constructor injection in controllers:**
ASP.NET Core automatically resolves constructor parameters when creating a controller. You never call `new EmployeeService()` inside a controller.

**Property injection:**
Not natively supported by the built-in container — but possible with third-party containers (Autofac, etc.). Constructor injection is preferred — it makes dependencies explicit and visible.

**Service locator anti-pattern:**
Injecting `IServiceProvider` into a class and calling `serviceProvider.GetService<T>()` inside methods is the "Service Locator" pattern — it hides dependencies and makes code harder to test. Avoid it.

**Captive dependency rule summary:**
| Service being injected | Into a Singleton | Into a Scoped | Into a Transient |
|---|---|---|---|
| Singleton | ✅ OK | ✅ OK | ✅ OK |
| Scoped | ❌ Captive dependency! | ✅ OK | ✅ OK |
| Transient | ✅ OK (but captured instance) | ✅ OK | ✅ OK |

---

## 3. Middleware Pipeline — In Depth

**What is Middleware?**
Middleware is a component that sits in the **HTTP request/response pipeline**. Each middleware:
1. Receives the request from the previous middleware
2. Does its work (logging, auth checking, compression, etc.)
3. Either calls the **next** middleware OR short-circuits and sends a response

**The pipeline is a chain of delegates.** Each `app.Use(...)` call wraps the remaining pipeline in a new delegate.

**Execution order is an onion:**
```
Request →
  [Middleware 1 — before]
    [Middleware 2 — before]
      [Middleware 3 — before]
        → Action executes
      [Middleware 3 — after]
    [Middleware 2 — after]
  [Middleware 1 — after]
← Response
```
Notice: the first middleware registered is the outermost layer — it runs first on request AND last on response.

**Correct pipeline order in ASP.NET Core:**
```
app.UseExceptionHandler()    ← MUST be first — wraps everything to catch exceptions
app.UseHsts()
app.UseHttpsRedirection()
app.UseStaticFiles()
app.UseRouting()
app.UseCors()                ← Before auth
app.UseAuthentication()      ← Reads token, populates HttpContext.User
app.UseAuthorization()       ← Checks [Authorize] — MUST be after Authentication
app.UseSession()
app.MapControllers()
```

**Why order matters:**
- If `UseAuthorization` is before `UseAuthentication`, the `User` isn't set yet — all requests appear anonymous
- If `UseExceptionHandler` is not first, exceptions thrown in authentication or routing aren't caught
- If `UseRouting` is not before `UseAuthorization`, the framework doesn't know which endpoint is being called, so endpoint-specific authorization policies can't be evaluated

**`app.Use` vs `app.Run` vs `app.Map`:**

| Method | Calls `next`? | Behaviour |
|---|---|---|
| `app.Use(async (ctx, next) => {...})` | ✅ Yes | Passes to next middleware. Has a "before" and "after" phase. |
| `app.Run(async ctx => {...})` | ❌ No | Terminal — pipeline ends here. Any `Use` after this is unreachable. |
| `app.Map("/path", app => {...})` | Branches | Creates a separate sub-pipeline for requests matching the path prefix. |
| `app.MapWhen(condition, app => {...})` | Branches | Branches based on any condition on `HttpContext`. |

**Custom middleware class (recommended for reuse):**
Instead of inline lambdas, encapsulate middleware logic in a class with `InvokeAsync(HttpContext context, RequestDelegate next)`. Register with `app.UseMiddleware<LoggingMiddleware>()`.

**Interview Answer:** *"Middleware is a chain of components that handle HTTP requests and responses. Each component can run logic before and after passing to the next one. The pipeline is configured in Program.cs and order matters critically — authentication must come before authorization. app.Use passes to the next middleware, app.Run is terminal and stops the pipeline, app.Map branches the pipeline for a specific path prefix."*

---

## 4. Model Binding + Data Annotations

**What is Model Binding?**
Model binding is the process where ASP.NET Core **automatically reads data from the incoming HTTP request and maps it to C# method parameters and objects**. You don't manually parse request bodies or query strings.

**Binding sources — in priority order:**

| Source Attribute | Where data comes from | Example |
|---|---|---|
| `[FromRoute]` | URL path segment | `GET /employees/42` → `int id = 42` |
| `[FromQuery]` | URL query string | `?page=2&size=10` → `int page, int size` |
| `[FromBody]` | Request body (JSON/XML) | POST body → `CreateEmployeeDto dto` |
| `[FromHeader]` | HTTP request header | `X-Api-Key: abc123` |
| `[FromForm]` | HTML form submission | `<form>` multipart/urlencoded data |
| `[FromServices]` | DI container | Inject a service directly into an action |

**With `[ApiController]`:**
- Simple types (int, string) → automatically from route/query
- Complex types (DTOs, objects) → automatically `[FromBody]`

**Model validation flow:**
1. Request arrives, model binder maps data to the action parameter
2. Validation attributes on the model properties are evaluated
3. Results go into `ModelState`
4. With `[ApiController]`: if `!ModelState.IsValid`, **400 is returned automatically** with all validation errors
5. Without `[ApiController]`: you must check `if (!ModelState.IsValid) return BadRequest(ModelState);`

**Data Annotations — validation attributes in detail:**

| Attribute | Validates | Example |
|---|---|---|
| `[Required]` | Not null, not empty string | `[Required] string Name` |
| `[StringLength(max)]` | Max character length | `[StringLength(100)] string Name` |
| `[MinLength(min)]` / `[MaxLength(max)]` | Collection / string length | — |
| `[Range(min, max)]` | Numeric range | `[Range(0, 150)] int Age` |
| `[EmailAddress]` | Valid email format | `[EmailAddress] string Email` |
| `[Phone]` | Valid phone number | — |
| `[Url]` | Valid URL | — |
| `[RegularExpression(pattern)]` | Regex match | `[RegularExpression(@"^[A-Z]")]` |
| `[Compare("OtherProp")]` | Must match another property | `[Compare("Password")] string Confirm` |
| `[CreditCard]` | Valid credit card format | — |

**DTO (Data Transfer Object) vs Entity:**
You should **never** bind directly to your EF Core entity in an API action. Use a separate **DTO class** for input (create/update requests) and output. Reasons:
1. **Security** — prevent over-posting (client setting fields you didn't intend to allow)
2. **Decoupling** — your API contract doesn't change when your DB schema changes
3. **Validation** — DTOs carry validation attributes; entities carry persistence attributes

---

## 5. Filters — In Depth

**What are Filters?**
Filters are components that run **at specific points in the ASP.NET Core request processing pipeline**, specifically around **action execution**. They implement cross-cutting concerns (logging, auth, caching) without polluting your action methods.

**The filter pipeline (order of execution):**
```
Request
  → Authorization Filter   (can short-circuit — return 401/403)
    → Resource Filter       (before model binding)
      → Model Binding
        → Action Filter     (OnActionExecuting)
          → Action method executes
        → Action Filter     (OnActionExecuted)
          → Result Filter   (OnResultExecuting)
            → Result executes (view rendering, JSON serialization)
          → Result Filter   (OnResultExecuted)
    → Resource Filter (after result)
  → (Exception Filter — catches exceptions from any of the above)
```

**Filter types in detail:**

| Filter | Interface | When it runs | Key use case |
|---|---|---|---|
| **Authorization** | `IAuthorizationFilter` | First — before everything | Custom auth logic not covered by `[Authorize]` |
| **Resource** | `IResourceFilter` | After auth, wraps model binding and result | Output caching, short-circuit before model binding |
| **Action** | `IActionFilter` | Before and after action method | Logging, input validation, timing |
| **Exception** | `IExceptionFilter` | When unhandled exception from action | Convert exceptions to `ProblemDetails` responses |
| **Result** | `IResultFilter` | Before and after result execution | Modify response headers, response transformation |

**IActionFilter — most commonly asked:**
```
OnActionExecuting  → runs before the action method body
Action method runs
OnActionExecuted   → runs after the action method body
```
Practical use cases:
- Log the action name, parameters, user, and timing
- Add a custom response header with execution time
- Validate a custom API version header
- Enforce business-level pre/post conditions

**IExceptionFilter:**
- Catches exceptions thrown by action methods (not by middleware)
- Can return a `ProblemDetails` response
- `context.ExceptionHandled = true` — marks exception as handled so middleware doesn't also catch it

**Scope — where you apply filters:**
| Scope | How | Effect |
|---|---|---|
| **Global** | `builder.Services.AddControllers(o => o.Filters.Add<LoggingFilter>())` | All actions |
| **Controller** | `[ServiceFilter(typeof(LoggingFilter))]` on class | All actions in that controller |
| **Action** | `[ServiceFilter(typeof(LoggingFilter))]` on method | That specific action only |

**`ServiceFilter` vs `TypeFilter`:**
- `[ServiceFilter(typeof(MyFilter))]` — filter is resolved from DI (supports constructor injection) ✅
- `[TypeFilter(typeof(MyFilter))]` — ASP.NET creates the filter, can pass constructor args
- `[MyFilter]` (inherits `Attribute` + `IActionFilter`) — no DI support, uses `IFilterFactory` for DI

**Interview Answer:** *"Filters are components that run at specific points around action execution — before/after model binding, before/after the action, when an exception occurs. The most common is IActionFilter which has OnActionExecuting (before) and OnActionExecuted (after). Use filters for cross-cutting concerns like logging and timing. The key difference from middleware: middleware handles the entire HTTP pipeline including static files, routing etc. Filters only operate within the MVC/Web API controller layer."*

---

## 6. Authentication & Authorization — In Depth

**The fundamental difference:**
- **Authentication**: Establishes **identity** — *who is this user?* Reads the JWT, cookie, or API key and says "this is user ID 42, email alice@example.com"
- **Authorization**: Establishes **permission** — *what can this user do?* Checks the authenticated user's roles and claims against what the endpoint requires

**Authentication schemes in ASP.NET Core:**
| Scheme | How it works | Used for |
|---|---|---|
| **JWT Bearer** | Client sends `Authorization: Bearer <token>` header | SPAs, mobile apps, microservices |
| **Cookie** | Browser stores encrypted cookie, sent automatically | Traditional web apps (MVC) |
| **API Key** | Custom header/query string with a secret key | Server-to-server, simple integrations |
| **OAuth 2.0 / OIDC** | Delegate to an identity provider (Azure AD, Google) | Enterprise SSO |

**How ASP.NET Core Authentication middleware works:**
1. `UseAuthentication()` middleware runs
2. It reads the configured scheme (e.g., JWT Bearer)
3. Finds the token in the `Authorization` header
4. Validates signature using the secret key
5. Validates expiry, issuer, audience claims
6. If valid → builds `ClaimsPrincipal` and sets `HttpContext.User`
7. If invalid → `HttpContext.User` is anonymous (unauthenticated)

**Authorization types:**

| Type | How | When to use |
|---|---|---|
| **Simple** | `[Authorize]` | Just needs to be authenticated |
| **Role-based** | `[Authorize(Roles = "Admin")]` | Role membership |
| **Policy-based** | `[Authorize(Policy = "SeniorOnly")]` | Complex rules — roles + claims + custom logic |
| **Resource-based** | `IAuthorizationService.AuthorizeAsync(user, resource, requirement)` | "Does this user own this specific record?" |
| **`[AllowAnonymous]`** | Overrides `[Authorize]` | Public endpoints |

**Claims-based identity:**
The `ClaimsPrincipal` (the authenticated user) carries a collection of **claims** — key-value pairs about the user:
- `ClaimTypes.NameIdentifier` → user ID
- `ClaimTypes.Email` → email
- `ClaimTypes.Role` → one claim per role
- Custom claims → `"department"`, `"employee_level"`, etc.

Claims are issued at login time (put into the JWT) and read on every request by the middleware.

**Policy-based authorization (recommended for complex rules):**
Policies are defined once in `Program.cs` and reused across controllers. A policy can require roles, specific claim values, or run custom logic through an `IAuthorizationRequirement` + `IAuthorizationHandler` pair.

---

## 7. JWT Token Flow — In Depth

**What is JWT?**
JSON Web Token — a compact, **self-contained** token that carries all the information needed to authenticate a user. The server doesn't need to look anything up in a database to validate a JWT — it just verifies the signature.

**Structure (3 parts, Base64URL encoded, separated by dots):**

| Part | Contains | Example |
|---|---|---|
| **Header** | Algorithm + type | `{ "alg": "HS256", "typ": "JWT" }` |
| **Payload** | Claims (user data) | `{ "sub": "42", "email": "a@b.com", "role": "Admin", "exp": 1740000000 }` |
| **Signature** | `HMAC_SHA256(base64(header) + "." + base64(payload), secretKey)` | Prevents tampering |

**The full authentication flow:**

```
Step 1 — Login:
  Client → POST /api/auth/login { username, password }
  Server: validates credentials against DB
  Server: creates JWT with claims (userId, email, roles)
  Server: signs with secret key (HMAC-SHA256 or RSA)
  Server → Client: { "token": "eyJ..." }

Step 2 — Subsequent requests:
  Client → GET /api/employees
  Headers: Authorization: Bearer eyJ...
  
  Server: UseAuthentication middleware intercepts
  Server: extracts token from Authorization header
  Server: validates signature (was it signed with our key?)
  Server: validates expiry (is exp > now?)
  Server: validates issuer and audience
  Server: populates HttpContext.User with claims from token
  
  UseAuthorization: checks [Authorize] on the endpoint
  If user is authenticated and has required role → action runs
  If not → 401 Unauthorized
```

**Standard JWT claims (registered claims):**
| Claim | Full name | Meaning |
|---|---|---|
| `sub` | Subject | User identifier |
| `iss` | Issuer | Who created the token (your API URL) |
| `aud` | Audience | Intended recipient |
| `exp` | Expiration | Unix timestamp — token expires at this time |
| `iat` | Issued At | When the token was created |
| `nbf` | Not Before | Token not valid before this time |

**Why JWT is stateless:**
The server does NOT store tokens. All user information is inside the token. The signature proves the token hasn't been tampered with. Any server with the secret key can validate any token — perfect for horizontal scaling (load balancers, microservices).

**Security considerations:**
| Concern | Mitigation |
|---|---|
| Token theft | Short expiry (15–60 min) + Refresh tokens |
| Secret key exposure | Store in Azure Key Vault / env vars, not appsettings.json |
| Replay attacks | Short expiry; optionally maintain a token blacklist for logout |
| Sensitive data in payload | Payload is Base64 (not encrypted) — don't put passwords/secrets in claims |

**Refresh token pattern:**
Access token: short-lived (15 min), stored in memory/session.
Refresh token: long-lived (7 days), stored in HttpOnly cookie (not accessible to JS → XSS protection), stored in DB.
When access token expires → client uses refresh token to get a new access token without re-login.

---

## 8. appsettings.json & IConfiguration

**What is it?**
`appsettings.json` is the default configuration file for ASP.NET Core applications. `IConfiguration` is the interface used to read configuration values anywhere in the app.

**Configuration sources (ASP.NET Core reads from many, in priority order):**
1. `appsettings.json` — base settings (lowest priority)
2. `appsettings.{Environment}.json` — environment-specific overrides
3. User Secrets (development only) — local machine, not committed to Git
4. Environment variables — deployment/container settings
5. Command-line arguments (highest priority — override everything)

**Environment-specific override mechanism:**
ASP.NET Core reads `ASPNETCORE_ENVIRONMENT` env variable. If it's `"Development"`, it loads and merges `appsettings.Development.json` on top of the base. Keys in the environment-specific file win.

**Accessing configuration:**

| Pattern | Use case |
|---|---|
| `IConfiguration["SectionName:Key"]` | Quick access, returns string |
| `IConfiguration.GetSection("Jwt").Get<JwtSettings>()` | Bind a section to a typed POCO |
| `builder.Services.Configure<JwtSettings>(config.GetSection("Jwt"))` | Bind + register in DI as `IOptions<JwtSettings>` |
| `IOptions<JwtSettings>` injected in constructor | Preferred in services |

**Options pattern (`IOptions<T>`) — the right way:**
Define a plain C# class (POCO) matching the JSON structure. Register it with `Configure<T>`. Inject `IOptions<T>` in your service. This is **strongly typed** — no magic strings, compile-time safety.

**Secrets management — critical:**
| Environment | Secret storage |
|---|---|
| Development | **User Secrets** (`dotnet user-secrets set "Jwt:Key" "..."`) — stored outside project folder, never in Git |
| CI/CD pipeline | **Pipeline secret variables** — injected as environment variables at runtime |
| Production (Azure) | **Azure Key Vault** — store keys/connection strings, App Service reads them as env vars |

**Never commit secrets to Git.** The moment a key is committed, it should be considered compromised — rotate it immediately. Use `.gitignore` to exclude `appsettings.*.json` files with real credentials.

---

## 9. EF Core — DbContext, DbSet & Change Tracker

**What is EF Core?**
EF Core is an **Object-Relational Mapper (ORM)** — it bridges C# objects (entities) and a relational database. You work with C# classes and LINQ; EF Core generates the SQL and maps results back to objects.

### DbContext — In Depth

**What it is:** The central class in EF Core — represents a **session/unit of work** with the database.

**What it manages:**
- **Connection lifecycle** — opens/closes DB connections (pooled automatically)
- **Change tracking** — tracks every entity loaded and what changed
- **Transaction management** — wraps `SaveChangesAsync` in a transaction by default
- **SQL generation** — translates LINQ to SQL
- **Concurrency** — handles optimistic concurrency with row version tokens

**Entity states (Change Tracker states):**
| State | Meaning | What SaveChanges does |
|---|---|---|
| `Added` | New object, not in DB yet | `INSERT` |
| `Modified` | Loaded from DB, properties changed | `UPDATE` |
| `Deleted` | Marked for deletion | `DELETE` |
| `Unchanged` | Loaded from DB, nothing changed | Nothing |
| `Detached` | Not tracked by this context | Nothing |

**How the Change Tracker works:**
When you `await _context.Employees.ToListAsync()`, EF Core:
1. Takes a **snapshot** of every loaded entity's values
2. Stores the entity in the Change Tracker as `Unchanged`

When you call `await _context.SaveChangesAsync()`, EF Core:
1. Compares current values of each tracked entity to its snapshot
2. For entities with changed values → marks as `Modified`, generates `UPDATE`
3. For new entities → generates `INSERT`
4. For deleted entities → generates `DELETE`
5. Wraps all SQL in a single transaction and executes

**`AsNoTracking()` — when and why:**
- Disables the Change Tracker for that query
- EF Core doesn't take snapshots → faster (less memory, less CPU)
- Use for **read-only** queries where you'll never call `SaveChanges` with that data
- `_context.Employees.AsNoTracking().ToListAsync()` — list page, report, dashboard

### DbSet\<T\>
Represents a **table** in the database. It's an `IQueryable<T>` — queries built on it are translated to SQL and run in the database.

---

## 10. Code-First Migrations — In Depth

**What are Migrations?**
Migrations are a **version control system for your database schema**. When you change your entity classes, you create a migration — EF Core generates C# code that describes the schema change. Applying the migration runs the corresponding SQL.

**Why migrations instead of SQL scripts:**
- **Version controlled** alongside code — schema change + code change in the same commit
- **Repeatable** — new team members run `database update` to sync their local DB
- **Reversible** — `Down()` method lets you roll back
- **Team-friendly** — each developer creates migrations locally, they're merged and applied in order

**Migration workflow:**
```
1. Modify entity class (add property, change type, add relationship)
2. dotnet ef migrations add "AddEmployeeSalaryColumn"
   → EF compares current model to last migration snapshot
   → Generates a .cs file with Up() and Down() methods
3. Review the generated migration file — always check it's correct
4. dotnet ef database update
   → EF applies all pending migrations in order
   → Records applied migrations in __EFMigrationsHistory table
```

**What a migration file contains:**
- `Up()` — the forward change: `migrationBuilder.AddColumn<decimal>("Salary", "Employees", ...)`
- `Down()` — the reverse: `migrationBuilder.DropColumn("Salary", "Employees")`
- A **model snapshot** is maintained in `Migrations/[Context]ModelSnapshot.cs` — EF diffs against this

**Managing migrations in a team:**
- Each developer creates their own migration — they stack on top of each other
- If two developers create conflicting migrations (both modified the same table), one must be rebased
- CI/CD pipelines apply migrations to staging/production automatically: `dotnet ef database update --connection "..."`

**Seeding data:**
EF Core supports data seeding in `OnModelCreating` via `modelBuilder.Entity<Role>().HasData(...)` — seed data is managed through migrations and applied automatically.

---

## 11. Navigation Properties & Relationships — In Depth

**What are Navigation Properties?**
Navigation properties are C# properties on an entity class that **reference related entities**. They tell EF Core about relationships and allow you to traverse related data in C#.

**Three relationship types:**

**One-to-Many (most common):**
```
Department has many Employees
→ Department.Employees = ICollection<Employee>
→ Employee.Department = Department  (reference navigation)
→ Employee.DepartmentId = int       (foreign key property)
```

**One-to-One:**
```
Employee has one Address
→ Employee.Address = Address        (reference navigation)
→ Address.Employee = Employee       (inverse navigation)
→ Address.EmployeeId = int          (foreign key — on the dependent side)
```

**Many-to-Many:**
```
Student enrolls in many Courses; Course has many Students
→ Student.Courses = ICollection<Course>
→ Course.Students = ICollection<Student>
→ EF Core 5+ creates join table automatically (StudentCourse)
→ Or define explicit join entity with extra columns (Enrollment with Grade)
```

**Cascade delete behaviour:**
| DeleteBehavior | What happens when parent is deleted |
|---|---|
| `Cascade` | Children are automatically deleted |
| `Restrict` | Exception thrown — must delete children first |
| `SetNull` | FK on children is set to NULL |
| `NoAction` | DB decides (usually error) |

**Choosing cascade vs restrict:**
- `Cascade` — when children have no meaning without the parent (Order → OrderItems)
- `Restrict` — when you want to prevent accidental deletions (Department → Employees — don't accidentally delete all employees when deleting a department)

---

## 12. Eager vs Lazy vs Explicit Loading — In Depth

**The core problem:** Related data is stored in a separate table. EF Core doesn't automatically load it — you must tell it how and when.

### Eager Loading — `.Include()`
**How:** Add `.Include()` to your query. EF Core performs a JOIN in the generated SQL and loads related data in the same query.

**When to use:**
- You **know you will need** the related data when you write the query
- Reporting, list views that show related data
- Best for performance — one SQL query instead of many

**Multiple levels:** `.Include(e => e.Department).ThenInclude(d => d.Manager)` — joins three tables in one query.

### Lazy Loading
**How:** Navigation properties are loaded automatically the first time you access them. Requires proxy classes (install `Microsoft.EntityFrameworkCore.Proxies`, configure in DbContext, mark nav properties `virtual`).

**The N+1 problem:**
```
Load 100 employees: SELECT * FROM Employees           ← 1 query
foreach (var emp in employees) {
    Console.WriteLine(emp.Department.Name);           ← 1 query per employee = 100 extra queries!
}
Total: 101 queries instead of 1 JOIN query → DISASTER for performance
```

**When lazy loading is acceptable:** Interactive UI where you genuinely don't know which related data will be needed until the user navigates to it.

**Rule of thumb for APIs:** Never use lazy loading in Web API projects. Always use eager loading or explicit loading.

### Explicit Loading
**How:** Load related data on demand using `_context.Entry(entity).Reference(e => e.Department).LoadAsync()`.

**When to use:** You loaded a single entity and need to conditionally load a specific navigation property based on business logic — gives you precise control.

**Summary decision:**
| Scenario | Use |
|---|---|
| List page that shows dept name with employees | Eager (`.Include`) |
| Loading one employee, maybe need dept | Explicit loading |
| Don't know upfront what related data is needed | Lazy (only if UI, never API) |

---

## 13. LINQ-to-Entities — How EF Core Translates LINQ to SQL

**What happens when you write EF LINQ:**
When you write `_context.Employees.Where(e => e.Salary > 50000)`, the lambda is NOT executed as C# code. Instead:
1. The compiler creates an **Expression Tree** — a data structure representing the code as data
2. EF Core's query provider **reads the expression tree**
3. It translates it into SQL: `SELECT * FROM Employees WHERE Salary > 50000`
4. SQL is sent to the database
5. Results are mapped back to `Employee` objects

**Why this matters — IQueryable vs IEnumerable:**
| | Interface | Where filtering runs | Performance (1M rows, 10 match) |
|---|---|---|---|
| EF LINQ query | `IQueryable<T>` | **Database** (SQL WHERE clause) | Fetches 10 rows ✅ |
| After `.ToList()` | `IEnumerable<T>` | **C# memory** | Fetches 1,000,000 rows ❌ |

**Common LINQ-to-SQL translations:**

| C# LINQ | Generated SQL |
|---|---|
| `.Where(e => e.DeptId == 1)` | `WHERE DepartmentId = 1` |
| `.OrderBy(e => e.Name)` | `ORDER BY Name ASC` |
| `.Skip(10).Take(5)` | `OFFSET 10 ROWS FETCH NEXT 5 ROWS ONLY` |
| `.Select(e => new { e.Name, e.Salary })` | `SELECT Name, Salary` (no SELECT *) |
| `.Include(e => e.Dept)` | `LEFT JOIN Departments` |
| `.GroupBy(e => e.DeptId)` | `GROUP BY DepartmentId` |
| `.Count()` | `SELECT COUNT(*)` |
| `.Any(e => e.Active)` | `SELECT CASE WHEN EXISTS (SELECT ...) THEN 1 ELSE 0 END` |

**What EF Core CANNOT translate (causes client-side evaluation or exception):**
- Calling custom C# methods inside LINQ that have no SQL equivalent
- Complex string operations not supported by the DB provider
- In EF Core 3.0+: throws an exception if it can't translate (instead of silently loading everything)

**Best practices:**
- Filter, sort, and page in `IQueryable` before calling `.ToList()` — everything in the database
- Use `Select` to project only needed columns — avoid `SELECT *`
- Use `AsNoTracking()` for read-only queries
- Use `Include` explicitly — never rely on lazy loading in API projects

---

## 📝 Revision Checklist

### Day 1 — Can you explain these without looking?
- [ ] 4 OOP pillars — give a real example for each
- [ ] Interface vs abstract class — when to choose each
- [ ] Value type vs reference type — what happens when you assign one to another
- [ ] What is a delegate? How do `Func` and `Action` relate?
- [ ] How does async/await work internally? Why does `.Result` cause deadlocks?
- [ ] What is deferred execution in LINQ? When does a LINQ query actually execute?
- [ ] Explain each SOLID principle in one sentence with a violation symptom
- [ ] What does CLR do? Explain the compilation pipeline.
- [ ] What are the 3 DI lifetimes? Why must DbContext be Scoped?

### Day 2 — Can you explain these without looking?
- [ ] What does `[ApiController]` add? Name all 4 behaviours.
- [ ] Why does middleware order matter? What happens if `UseAuthentication` comes after `UseAuthorization`?
- [ ] `app.Use` vs `app.Run` — what is the difference?
- [ ] What is a DTO and why should you use it instead of entities in API actions?
- [ ] Explain each filter type and when you'd use each
- [ ] Authentication vs authorization — what is each one doing?
- [ ] JWT structure — what are the 3 parts? Why is JWT stateless?
- [ ] What is the N+1 problem? How does `.Include()` fix it?
- [ ] What is the EF Core Change Tracker? What does `AsNoTracking()` do?
- [ ] What happens when you write a LINQ query against a `DbSet`? (Expression trees → SQL)
- [ ] What are Code-First migrations? What do `Up()` and `Down()` do?

---

## 🔗 Cross-References — Deeper Dives

| Topic | Go to |
|---|---|
| CLR, async/await, Extension Methods, Reflection | [Module 1](./Module1_DotNET8_CSharp12.md) — Section 1.3 |
| SOLID with full code examples | [Module 2](./Module2_SOLID_Principles.md) |
| IEnumerable vs IQueryable, Fluent API | [Module 4](./Module4_EFCore_Dapper.md) — Sections 4.13, 4.14 |
| DI lifetimes with DbContext race condition | [Module 5](./Module5_ASPNETCore8.md) — Section 5.11 |
| app.Use vs app.Run vs app.Map | [Module 5](./Module5_ASPNETCore8.md) — Section 5.12 |
| JWT full implementation | [Module 6](./Module6_WebAPI.md) — Section 6.10 |
| [ApiController] detail | [Module 6](./Module6_WebAPI.md) — Section 6.15 |
| Role-Based Authorization | [Module 6](./Module6_WebAPI.md) — Section 6.16 |

---

_[← Back to README](./README.md)_
