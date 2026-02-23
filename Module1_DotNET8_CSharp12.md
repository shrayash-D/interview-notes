# 📌 MODULE 1: .NET 8 AND C# 12

---

## 1.1 Introduction to .NET 8

### What is .NET?

> **.NET** is a **free, open-source developer platform** by Microsoft for building many types of applications — web, desktop, mobile, cloud, gaming, IoT, etc.

**In simple words:** Think of .NET as a toolbox + engine. You write code in C#, and .NET provides the libraries, runtime, and tools to compile and run that code on any operating system — Windows, Mac, or Linux.

**Interview Answer:** ".NET is a cross-platform, open-source framework by Microsoft that provides tools and libraries to build all kinds of applications like web APIs, desktop apps, and cloud services."

---

### Evolution: .NET Framework → .NET 6 → .NET 8

| Version                   | Key Point                                                  |
| ------------------------- | ---------------------------------------------------------- |
| **.NET Framework** (2002) | Windows-only, runs on CLR (Common Language Runtime)        |
| **.NET Core** (2016)      | Cross-platform rewrite, open-source, faster                |
| **.NET 5** (2020)         | Unified .NET Core + .NET Framework into one platform       |
| **.NET 6** (2021)         | LTS (Long Term Support), Minimal APIs introduced           |
| **.NET 7** (2022)         | Performance improvements, short-term support               |
| **.NET 8** (2023)         | **LTS**, best performance yet, Native AOT, improved Blazor |

**Why did Microsoft move from .NET Framework to .NET Core?**
Because .NET Framework was Windows-only and tightly coupled to IIS. The industry needed cross-platform support (Linux servers, Docker containers). .NET Core was a ground-up rewrite that runs everywhere.

**Interview Answer:** "Microsoft evolved from the Windows-only .NET Framework to the cross-platform .NET Core, then unified everything into a single .NET platform starting from .NET 5. .NET 8 is the latest LTS version with major performance improvements."

---

### Key Features in .NET 8

| Feature                            | Simple Explanation                                                        |
| ---------------------------------- | ------------------------------------------------------------------------- |
| **Native AOT**                     | App compiled to machine code at build time → faster startup, smaller size |
| **Performance Improvements**       | Up to 20% faster than .NET 7 in many benchmarks                           |
| **Improved Blazor**                | Unified full-stack web UI with server + client rendering                  |
| **Minimal APIs**                   | Lighter-weight APIs with less code                                        |
| **Garbage Collector Improvements** | Better memory management                                                  |
| **Frozen Collections**             | `FrozenDictionary`, `FrozenSet` — read-only, optimized for fast lookup    |

---

## 1.2 C# 12 Features

### Primary Constructors

**What it is:** Instead of writing a full constructor body to assign fields, you can define constructor parameters directly in the class declaration.

**Why it matters:** Removes boilerplate. A class that just needs to receive and hold dependencies no longer needs 4–5 lines of repetitive assignment code.

```csharp
// ❌ Old way (C# 11 and before) — too much boilerplate
public class Person
{
    private string _name;
    private int _age;

    public Person(string name, int age)
    {
        _name = name;
        _age = age;
    }
}

// ✅ New way (C# 12) — cleaner
public class Person(string name, int age)
{
    public string Name => name;
    public int Age => age;
}
```

**Interview Answer:** "Primary constructors let you define constructor parameters directly on the class declaration, reducing boilerplate. The parameters are available throughout the class body."

---

### Collection Expressions

**What it is:** A new unified `[ ]` syntax to create any kind of collection — arrays, lists, spans.

**Why it matters:** Previously you needed `new int[] { }` or `new List<string> { }`. Now one syntax works for all.

```csharp
// ❌ Old way
int[] numbers = new int[] { 1, 2, 3 };
List<string> names = new List<string> { "Alice", "Bob" };

// ✅ New way (C# 12)
int[] numbers = [1, 2, 3];
List<string> names = ["Alice", "Bob"];

// Spread operator (..) to combine collections
int[] first = [1, 2, 3];
int[] second = [4, 5, 6];
int[] combined = [..first, ..second]; // [1, 2, 3, 4, 5, 6]
```

---

### Default Lambda Parameters

**What it is:** Lambda expressions can now have default values for their parameters, just like regular methods.

```csharp
var greet = (string name = "World") => $"Hello, {name}!";
Console.WriteLine(greet());        // "Hello, World!"
Console.WriteLine(greet("Alice")); // "Hello, Alice!"
```

---

## 1.3 C# Core Concepts

---

### CLR — Common Language Runtime

**What is CLR?**
CLR is the **execution engine** of .NET. It is the layer that actually runs your compiled code. When you build a C# project, the compiler does NOT produce machine code directly — it produces **Intermediate Language (IL)**, also called MSIL or CIL. The CLR then takes that IL and compiles it to native machine code at runtime using **JIT (Just-In-Time) compilation**.

**Compilation flow:**

```
Your C# Code (.cs)
    ↓  C# Compiler (csc / Roslyn)
IL / MSIL (.dll / .exe)
    ↓  CLR's JIT Compiler (at runtime)
Native Machine Code (runs on CPU)
```

**What CLR manages for you:**

| CLR Responsibility          | What It Does                                                  |
| --------------------------- | ------------------------------------------------------------- |
| **JIT Compilation**         | Converts IL → native machine code at runtime                  |
| **Garbage Collection (GC)** | Automatically finds and frees unused objects from heap memory |
| **Type Safety**             | Prevents illegal casts and memory access errors               |
| **Exception Handling**      | Unified exception model across all .NET languages             |
| **Thread Management**       | Creates, schedules, and synchronizes threads                  |
| **Security**                | Code Access Security, sandboxing                              |

**What is JIT?**
JIT (Just-In-Time) compilation means code is compiled to native code the **first time a method is called**. After that, the native version is cached. This is why .NET apps can feel slightly slow on first request and faster afterwards.

**What is Garbage Collection?**
GC automatically detects objects in heap memory that are no longer referenced and frees that memory. It uses a **generational model**:

- **Gen 0** — Short-lived objects (most objects die here, cheapest to collect)
- **Gen 1** — Medium-lived objects (survived Gen 0 collection)
- **Gen 2** — Long-lived objects (singletons, static data, expensive to collect)

**CTS vs CLS:**

- **CTS (Common Type System)** — Defines all types that .NET supports. Ensures a C# `int` and a VB.NET `Integer` are actually the same type at runtime.
- **CLS (Common Language Specification)** — A subset of CTS that all .NET languages must support so they can interoperate safely.

**Interview Answer:** "CLR is the execution engine of .NET. C# compiles to Intermediate Language (IL), and the CLR's JIT compiler converts IL to native machine code at runtime. CLR also handles garbage collection, exception handling, type safety, and thread management. The GC uses a generational model — Gen 0 for short-lived objects, Gen 2 for long-lived ones."

---

### async / await

**What is it?**
`async`/`await` is a language feature that lets you write **non-blocking** asynchronous code in a way that reads exactly like synchronous code — no callbacks, no manual thread management.

**The core idea:**
When your code hits `await`, it does NOT block the thread. Instead, the thread is **released back to the thread pool** to handle other requests. When the awaited operation completes (e.g., DB query finishes), a thread picks up execution from where it left off.

**Why does this matter for web APIs?**
A synchronous web server has a fixed number of threads. Each blocked thread (waiting on DB, network) means one fewer thread to handle new requests. With async/await, threads are never blocked waiting — they go back to the pool. This is why async APIs can handle thousands of concurrent requests with the same hardware.

**The state machine:**
When you mark a method `async`, the compiler transforms it into a **state machine** behind the scenes. Each `await` point becomes a "state" that the machine can pause at and resume from later.

| Rule                             | Detail                                                           |
| -------------------------------- | ---------------------------------------------------------------- |
| Return type                      | `Task` (no return value) or `Task<T>` (returns T)                |
| `async void`                     | Only for event handlers — errors are swallowed silently, avoid   |
| Never use `.Result` or `.Wait()` | Blocks the thread, causes deadlocks in ASP.NET Core              |
| Naming convention                | End async methods with `Async` (e.g., `GetEmployeeAsync`)        |
| Run parallel tasks               | Use `Task.WhenAll(t1, t2)` to fire multiple tasks simultaneously |

**Why does `.Result` cause a deadlock?**
ASP.NET Core has a synchronization context. When you call `.Result`, the current thread blocks waiting for the task. Meanwhile, the task's continuation is waiting for the same thread to become free. Both wait for each other → deadlock.

```csharp
// ✅ CORRECT — thread released while waiting for DB/network
public async Task<Employee> GetEmployeeAsync(int id)
{
    return await _context.Employees.FindAsync(id);  // Thread released here
}

// ❌ WRONG — blocks the thread, can cause deadlock
public Employee GetEmployee(int id)
{
    return _context.Employees.FindAsync(id).Result;  // Danger!
}

// Run multiple tasks simultaneously (not sequentially)
public async Task<(List<Employee>, List<Department>)> GetAllAsync()
{
    var empTask  = _context.Employees.ToListAsync();
    var deptTask = _context.Departments.ToListAsync();
    await Task.WhenAll(empTask, deptTask);  // Both run at the same time
    return (empTask.Result, deptTask.Result);
}
```

**Interview Answer:** "When `await` is hit, the current thread is released back to the thread pool while the I/O operation completes. No thread sits idle. For web APIs, this means one server can handle thousands of concurrent requests because no threads are blocked. Never use `.Result` or `.Wait()` — they block the current thread and can deadlock because the continuation is waiting for that same thread."

---

### Types of Classes in C#

**Why do different class types exist?**
Not all classes serve the same purpose. C# gives you modifiers to express your design intent clearly — forcing subclasses to provide implementations (abstract), preventing misuse through inheritance (sealed), grouping utility methods (static), or splitting generated code from hand-written code (partial).

| Class Type   | Key Rule                                                           | When to Use                                                                     |
| ------------ | ------------------------------------------------------------------ | ------------------------------------------------------------------------------- |
| **abstract** | Cannot be instantiated; subclasses must implement abstract members | Base types with shared behavior — `Animal`, `Shape`, `Repository`               |
| **sealed**   | Cannot be inherited from                                           | Security-sensitive classes, performance optimization, final implementations     |
| **static**   | No instances at all; only static members                           | Utility/helper classes — `Math`, `Console`, `StringHelper`                      |
| **partial**  | Definition split across multiple files; merged at compile time     | Code generators (EF scaffolding, WinForms designer) alongside hand-written code |

**Abstract class vs Interface — when to use which:**

| Aspect                | Abstract Class                                        | Interface                                 |
| --------------------- | ----------------------------------------------------- | ----------------------------------------- |
| Can have fields/state | ✅ Yes                                                | ❌ No (only properties)                   |
| Can have constructors | ✅ Yes                                                | ❌ No                                     |
| Implementation        | ✅ Can have concrete methods                          | ✅ Default methods (C# 8+), but limited   |
| Inheritance           | Single only                                           | Multiple interfaces allowed               |
| Relationship          | "is-a" — Dog **is an** Animal                         | "can-do" — Dog **implements** ISwimmable  |
| Use when              | Sharing code + enforcing contract among related types | Defining a capability for unrelated types |

```csharp
// abstract — forces subclasses to implement Sound()
public abstract class Animal
{
    public abstract string Sound();   // Must override in subclass
    public void Breathe() { }         // Shared concrete method
}

// sealed — nobody can extend this further
public sealed class FinalImpl : Animal
{
    public override string Sound() => "...";
}

// static — no instances, only utility methods
public static class MathHelper
{
    public static int Square(int x) => x * x;
}

// partial — split across files (common with generated code)
// File1.cs
public partial class OrderProcessor { public void Validate() { } }
// File2.cs
public partial class OrderProcessor { public void Save() { } }
```

**Interview Answer:** "Abstract classes define a base contract and can share implementation — subclasses must implement the abstract members. Sealed prevents inheritance — `string` in .NET is sealed. Static classes hold utility methods with no instances. Partial lets generated code and developer code live in separate files but compile as one class."

---

### `const` vs `readonly`

**What is the difference?**
Both are used for values that shouldn't change, but they differ in **when** the value is determined.

- `const` — value is baked in at **compile time**. The compiler replaces every use of the constant with its literal value in the IL.
- `readonly` — value is set at **runtime** (in the constructor or at declaration). The object is created first, then the value is assigned.

| Aspect                | `const`                      | `readonly`                                   |
| --------------------- | ---------------------------- | -------------------------------------------- |
| When set              | Compile-time only            | Runtime (constructor or inline)              |
| Implicitly static?    | ✅ Yes                       | ❌ No (can be instance or `static readonly`) |
| Allowed types         | Primitives and `string` only | Any type including objects                   |
| Changeable after set? | Never                        | Only in constructor                          |

**Practical rule:** Use `const` for true compile-time constants (Pi, AppVersion string). Use `readonly` for values that are determined at startup (config values, connection strings, initialized objects).

```csharp
public class Config
{
    public const double Pi = 3.14159;           // Compile-time — always the same value
    public const string AppName = "MyApp";

    public readonly string ConnectionString;    // Set once at runtime
    public static readonly DateTime AppStart = DateTime.UtcNow;  // Static readonly

    public Config(string connStr)
    {
        ConnectionString = connStr;             // Allowed in constructor ✅
        // ConnectionString = "other";          // ❌ Cannot reassign after constructor
    }
}
```

**Interview Answer:** "`const` is a compile-time constant — the value is embedded directly into the IL. `readonly` is a runtime constant — it's set in the constructor and cannot change after that. `const` only works for primitive types and strings. `readonly` can be any type, including objects."

---

### Extension Methods

**What is it?**
An extension method lets you **add a new method to an existing type** without modifying the type's source code, without subclassing it, and without any recompilation of the original type.

**Why is this powerful?**
You can extend `string`, `IEnumerable<T>`, or any third-party class that you can't modify. In fact, **all LINQ methods** (`Where`, `Select`, `OrderBy`, etc.) are extension methods defined on `IEnumerable<T>`.

**Rules for writing extension methods:**

1. Must be in a `static` class
2. The method must be `static`
3. The first parameter must use the `this` keyword before the type being extended

```csharp
public static class StringExtensions
{
    // Adds .IsNullOrEmpty() to every string in the app
    public static bool IsNullOrEmpty(this string value)
        => string.IsNullOrEmpty(value);

    // Adds .ToCamelCase() to every string
    public static string ToCamelCase(this string value)
        => string.IsNullOrEmpty(value) ? value
           : char.ToLower(value[0]) + value[1..];
}

// Used as if they were built-in string methods
string name = "Hello";
bool empty = name.IsNullOrEmpty();          // ✅ Feels like a native method
string camel = "MyVariable".ToCamelCase();  // → "myVariable"
```

**Interview Answer:** "Extension methods let you add methods to existing types without modifying them. They must be in a static class, be static methods, and use `this` before the first parameter. LINQ is entirely built from extension methods on `IEnumerable<T>` — `Where`, `Select`, `OrderBy` are all extension methods."

---

### Reflection

**What is it?**
Reflection is the ability to **inspect and interact with types at runtime** — reading their properties, methods, and attributes; creating instances dynamically; invoking methods — all without knowing the type at compile time.

**When is reflection used?**
You rarely write reflection code directly, but the frameworks you use do it constantly:

| Framework / Tool     | How It Uses Reflection                                  |
| -------------------- | ------------------------------------------------------- |
| **EF Core**          | Reads entity class properties to map them to DB columns |
| **ASP.NET Core DI**  | Reads constructor parameters to resolve dependencies    |
| **System.Text.Json** | Reads/writes properties by name at runtime              |
| **xUnit / NUnit**    | Discovers `[Fact]` / `[Test]` methods and runs them     |
| **AutoMapper**       | Maps matching property names between objects            |
| **[ApiController]**  | Reads attributes on controller to configure behavior    |

**Downside:** Reflection is slower than direct code because it bypasses compile-time optimizations and uses late binding. Avoid it in performance-critical hot paths.

```csharp
// Inspect a type at runtime
Type type = typeof(Employee);

// Get all public properties
PropertyInfo[] props = type.GetProperties();
foreach (var p in props)
    Console.WriteLine($"{p.Name}: {p.PropertyType.Name}");

// Create an instance dynamically (no compile-time knowledge of the type)
object emp = Activator.CreateInstance(typeof(Employee));

// Set a property value dynamically
PropertyInfo nameProp = type.GetProperty("Name");
nameProp.SetValue(emp, "John Doe");

// Invoke a method dynamically
MethodInfo method = type.GetMethod("GetFullName");
var result = method.Invoke(emp, null);
```

**Interview Answer:** "Reflection lets you inspect types, properties, and methods at runtime and invoke them dynamically. You don't use it directly often, but frameworks like EF Core, ASP.NET Core DI, and JSON serializers rely on it heavily. The downside is performance — reflection bypasses compile-time optimizations so it's slower than direct calls."

---

### Delegates, Func\<T\>, Action\<T\>

**What is a delegate?**
A delegate is a **type-safe function pointer** — a variable that holds a reference to a method. You can pass methods as arguments, store them in variables, and invoke them later.

**Why does this matter?**
Delegates are the foundation of events, callbacks, LINQ, and async programming in .NET. `Func<T>` and `Action<T>` are built-in generic delegate types that remove the need to declare your own delegate type for common patterns.

| Type               | Signature                  | Returns | Example Use                                        |
| ------------------ | -------------------------- | ------- | -------------------------------------------------- |
| `Action`           | No parameters              | void    | `Action log = () => Console.WriteLine("hi")`       |
| `Action<T>`        | One parameter              | void    | `Action<string> print = s => Console.WriteLine(s)` |
| `Func<T>`          | No parameters              | T       | `Func<int> getAge = () => 30`                      |
| `Func<T, TResult>` | One param, returns TResult | TResult | `Func<int,int> square = x => x * x`                |
| `Predicate<T>`     | One param                  | bool    | `Predicate<int> isEven = x => x % 2 == 0`          |

**Multicast delegates:**
A single delegate variable can hold references to **multiple methods**. When invoked, all methods are called in order. This is the foundation of C# events.

```csharp
// Custom delegate declaration
public delegate int MathOperation(int a, int b);

// Assign methods to delegates
MathOperation add = (a, b) => a + b;
MathOperation multiply = (a, b) => a * b;

Console.WriteLine(add(3, 4));      // 7
Console.WriteLine(multiply(3, 4)); // 12

// Func<T> — built-in, no custom declaration needed
Func<int, int, int> addFunc = (a, b) => a + b;
Console.WriteLine(addFunc(3, 4)); // 7

// Action<T> — same but returns void
Action<string> log = message => Console.WriteLine($"LOG: {message}");
log("User logged in");

// Multicast delegate — holds multiple methods
Action notify = () => Console.WriteLine("Email sent");
notify += () => Console.WriteLine("SMS sent");
notify += () => Console.WriteLine("Push notification sent");
notify(); // Calls all three
```

**Interview Answer:** "A delegate is a type-safe reference to a method — like a function pointer. `Func<T>` is a built-in delegate that returns a value; `Action<T>` is a built-in delegate that returns void. They're used everywhere in LINQ (the predicates you pass to `Where`, `Select` are `Func<T>` delegates). Multicast delegates can hold multiple method references, which is the basis of C# events."

---

### LINQ — Language Integrated Query

**What is LINQ?**
LINQ is a set of **extension methods** on `IEnumerable<T>` (and `IQueryable<T>`) that let you query and transform any collection using a consistent syntax — whether it's an in-memory list, a database via EF Core, or XML.

**Two syntaxes:**

- **Method syntax** (recommended): `employees.Where(e => e.Salary > 50000).OrderBy(e => e.Name)`
- **Query syntax** (SQL-like): `from e in employees where e.Salary > 50000 orderby e.Name select e`

Both compile to the same thing. Method syntax is more common in real codebases.

**Deferred execution:**
Most LINQ operators don't execute immediately. They build up a **query expression**. The query only runs when you force evaluation — with `.ToList()`, `.ToArray()`, `foreach`, `.Count()`, `.FirstOrDefault()` etc.

**Why deferred execution matters:**
You can chain multiple `.Where()` and `.Select()` calls — they all compose into one single efficient query. With EF Core, this means one SQL query is generated instead of multiple round trips.

| LINQ Operator                     | What It Does               | SQL Equivalent             |
| --------------------------------- | -------------------------- | -------------------------- |
| `Where`                           | Filter elements            | `WHERE`                    |
| `Select`                          | Project/transform elements | `SELECT`                   |
| `OrderBy` / `OrderByDescending`   | Sort                       | `ORDER BY ASC/DESC`        |
| `GroupBy`                         | Group elements             | `GROUP BY`                 |
| `Join`                            | Join two collections       | `INNER JOIN`               |
| `FirstOrDefault`                  | First match or null        | `SELECT TOP 1`             |
| `Count`                           | Total elements             | `COUNT(*)`                 |
| `Sum` / `Average` / `Max` / `Min` | Aggregation                | `SUM`, `AVG`, `MAX`, `MIN` |
| `Skip` / `Take`                   | Pagination                 | `OFFSET` / `FETCH NEXT`    |
| `Any`                             | At least one match?        | `EXISTS`                   |
| `All`                             | All match?                 | `NOT EXISTS (WHERE NOT)`   |
| `Distinct`                        | Remove duplicates          | `DISTINCT`                 |
| `Include` (EF Core)               | Eager load related data    | `JOIN`                     |

```csharp
var employees = new List<Employee> { ... };

// Filter + Sort + Project
var result = employees
    .Where(e => e.Salary > 50000)          // Filter
    .OrderBy(e => e.Name)                  // Sort
    .Select(e => new { e.Name, e.Salary }) // Project only needed fields
    .ToList();                             // Execute now

// Grouping
var byDept = employees
    .GroupBy(e => e.Department)
    .Select(g => new { Dept = g.Key, Count = g.Count(), Avg = g.Average(e => e.Salary) });

// Pagination
var page2 = employees
    .OrderBy(e => e.Name)
    .Skip(10)   // Skip first 10
    .Take(10)   // Take next 10
    .ToList();

// FirstOrDefault — returns null if not found (safe)
var emp = employees.FirstOrDefault(e => e.Id == 5);

// Any — does at least one match?
bool hasHighEarners = employees.Any(e => e.Salary > 100000);
```

**Interview Answer:** "LINQ provides a unified way to query any collection — in-memory lists or databases — using the same operators. It uses deferred execution: the query doesn't run until you call `.ToList()`, `foreach`, etc. With EF Core, LINQ expressions are translated into SQL at runtime, so `Where` and `Select` before `.ToList()` become part of the SQL query instead of loading everything into memory."

---

## 📝 Quick Recap

| Feature                     | Version      | One-liner                                   |
| --------------------------- | ------------ | ------------------------------------------- |
| Primary Constructors        | C# 12        | Constructor params on class declaration     |
| Collection Expressions `[]` | C# 12        | Unified syntax to create any collection     |
| Default Lambda Params       | C# 12        | Lambdas can have default values             |
| Native AOT                  | .NET 8       | Compile to machine code at build time       |
| Frozen Collections          | .NET 8       | Read-only, fast-lookup collections          |
| CLR                         | .NET Runtime | Runs IL via JIT, manages GC + exceptions    |
| async/await                 | C# 5+        | Non-blocking code, releases thread on await |
| Delegates/Func/Action       | C#           | Type-safe function references               |
| LINQ                        | C# 3+        | Query any collection, deferred execution    |

---

_[← Back to README](./README.md)_
