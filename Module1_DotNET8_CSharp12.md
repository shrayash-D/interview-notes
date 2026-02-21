# 📌 MODULE 1: .NET 8 AND C# 12

---

## 1.1 Introduction to .NET 8

### What is .NET?

> **.NET** is a **free, open-source developer platform** by Microsoft for building many types of applications — web, desktop, mobile, cloud, gaming, IoT, etc.

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

**Interview Answer:** "Microsoft evolved from the Windows-only .NET Framework to the cross-platform .NET Core, then unified everything into a single .NET platform starting from .NET 5. .NET 8 is the latest LTS version with major performance improvements."

---

### Key Features and Improvements in .NET 8

| Feature                                    | Simple Explanation                                                                |
| ------------------------------------------ | --------------------------------------------------------------------------------- |
| **Native AOT (Ahead-of-Time Compilation)** | App is compiled to machine code at build time → faster startup, smaller size      |
| **Performance Improvements**               | Up to 20% faster than .NET 7 in many benchmarks                                   |
| **Improved Blazor**                        | Unified full-stack web UI with server + client rendering                          |
| **Minimal APIs Enhancements**              | Easier to build lightweight APIs with less code                                   |
| **Garbage Collector Improvements**         | Better memory management                                                          |
| **JSON Improvements**                      | Faster System.Text.Json serialization                                             |
| **Time Abstraction**                       | New `TimeProvider` class makes testing time-based code easier                     |
| **Frozen Collections**                     | `FrozenDictionary`, `FrozenSet` — read-only collections optimized for fast lookup |

**Example — Frozen Collection:**

```csharp
// Frozen collections are optimized for read-heavy scenarios
FrozenDictionary<string, int> ages = new Dictionary<string, int>
{
    { "Alice", 30 },
    { "Bob", 25 }
}.ToFrozenDictionary();

int age = ages["Alice"]; // Super fast lookup!
```

---

## 1.2 C# 12 Features

### New Features in C# 12

#### 1️⃣ Primary Constructors (for Classes & Structs)

> You can now add constructor parameters directly in the class declaration — less boilerplate code.

```csharp
// ❌ Old way (C# 11 and before)
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

// ✅ New way (C# 12)
public class Person(string name, int age)
{
    public string Name => name;
    public int Age => age;
}
```

**Interview Answer:** "Primary constructors let you define constructor parameters directly on the class/struct declaration, reducing boilerplate code. The parameters are available throughout the class body."

---

#### 2️⃣ Collection Expressions

> A simpler, unified syntax to create collections using `[ ]`.

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

#### 3️⃣ Default Lambda Parameters

> Lambda expressions can now have default parameter values.

```csharp
// ✅ C# 12
var greet = (string name = "World") => $"Hello, {name}!";

Console.WriteLine(greet());        // "Hello, World!"
Console.WriteLine(greet("Alice")); // "Hello, Alice!"
```

---

#### 4️⃣ Alias Any Type

> You can now use `using` alias for ANY type — tuples, arrays, generics, etc.

```csharp
// ✅ C# 12
using Point = (int X, int Y);
using NumberList = System.Collections.Generic.List<int>;

Point p = (10, 20);
NumberList nums = [1, 2, 3];
```

---

#### 5️⃣ Inline Arrays

> Fixed-size arrays allocated on the stack for high-performance scenarios.

```csharp
[System.Runtime.CompilerServices.InlineArray(10)]
public struct Buffer
{
    private int _element;
}
```

---

#### 6️⃣ Interceptors (Experimental)

> Allows rerouting method calls to different implementations at compile time. Used mainly by source generators.

---

**Interview Answer:** "C# 12 introduces primary constructors for classes, collection expressions with a simpler `[]` syntax, default lambda parameters, and the ability to alias any type. These features reduce boilerplate and make code cleaner."

---

## 1.3 C# Core Concepts

### CLR — Common Language Runtime

> The **CLR** is the execution engine of the .NET runtime. It takes your compiled code (IL) and runs it on the machine.

**Compilation flow:**

```
Source Code (.cs) → C# Compiler → MSIL / CIL (.dll) → CLR (JIT Compiler) → Native Machine Code
```

| CLR Responsibility     | What It Does                                         |
| ---------------------- | ---------------------------------------------------- |
| **JIT Compilation**    | Converts IL → native machine code at runtime         |
| **Garbage Collection** | Automatically frees unused heap memory               |
| **Type Safety**        | Prevents illegal memory access / invalid type casts  |
| **Exception Handling** | Common exception model across all .NET languages     |
| **Thread Management**  | Manages thread creation, scheduling, synchronization |
| **Security**           | Code Access Security, sandboxing                     |

**Interview Answer:** _"CLR is the execution engine of .NET. C# compiles to Intermediate Language (IL), and the CLR's JIT compiler converts IL to native machine code at runtime. CLR also handles garbage collection, exception handling, type safety, and thread management."_

---

### async / await

> `async`/`await` enables **non-blocking** asynchronous code that reads like synchronous code. The key point: the **thread is released** while waiting — it doesn't sit idle.

```csharp
// ✅ CORRECT — thread released while waiting for DB/network
public async Task<Employee> GetEmployeeAsync(int id)
{
    return await _context.Employees.FindAsync(id);  // Thread released here
}

// ❌ WRONG — .Result/.Wait() blocks the thread and can DEADLOCK
public Employee GetEmployee(int id)
{
    return _context.Employees.FindAsync(id).Result;  // Blocks thread!
}

// Run multiple tasks in parallel
public async Task<(List<Employee>, List<Department>)> GetAllAsync()
{
    var empTask  = _context.Employees.ToListAsync();
    var deptTask = _context.Departments.ToListAsync();
    await Task.WhenAll(empTask, deptTask);          // Both run simultaneously
    return (empTask.Result, deptTask.Result);
}
```

| Rule                        | Detail                                                 |
| --------------------------- | ------------------------------------------------------ |
| Return type                 | `Task` (void) or `Task<T>` (with return value)         |
| `async void`                | Only for event handlers — no error propagation         |
| Never use `.Result`/`.Wait` | Risk of deadlock in ASP.NET Core                       |
| Suffix convention           | Method names end in `Async` (e.g., `GetEmployeeAsync`) |
| Parallel tasks              | Use `Task.WhenAll(t1, t2)` to run tasks simultaneously |

**Interview Answer:** _"When the `await` keyword is hit, the thread is released back to the thread pool while the I/O completes. For Web APIs, async = thousands of concurrent requests. Sync = ~100 before thread pool is exhausted. Never use `.Result` or `.Wait()` — they block the thread and can cause deadlocks."_

---

### Types of Classes in C#

| Class Type   | Key Rule                                                  | Example / Use Case                             |
| ------------ | --------------------------------------------------------- | ---------------------------------------------- |
| **abstract** | Cannot instantiate; force subclasses to implement members | `Animal` → `Dog`, `Cat`                        |
| **sealed**   | Cannot inherit from it                                    | `string` is sealed; security-sensitive classes |
| **static**   | No instances; only static members                         | `Math`, `Console`, utility/helper classes      |
| **partial**  | Split across multiple files; combined at compile time     | Code-behind in WinForms/WPF, generated code    |

```csharp
// abstract — template for subclasses
public abstract class Animal
{
    public abstract string Sound();      // Must override
    public void Breathe() { }           // Concrete method — shared
}

// sealed — no further inheritance allowed
public sealed class FinalImpl : Animal
{
    public override string Sound() => "...";
}

// static — utility class
public static class MathHelper
{
    public static int Square(int x) => x * x;
}

// partial — split across files
// File1.cs
public partial class OrderProcessor
{
    public void Validate() { }
}
// File2.cs
public partial class OrderProcessor
{
    public void Save() { }
}
```

**Abstract class vs Interface:**

| Aspect         | Abstract Class                | Interface                    |
| -------------- | ----------------------------- | ---------------------------- |
| Instantiate    | ❌ No                         | ❌ No                        |
| State / Fields | ✅ Yes                        | ❌ No (only properties)      |
| Implementation | ✅ Can have concrete methods  | ✅ Default methods (C# 8+)   |
| Inheritance    | Single (one base class)       | Multiple interfaces allowed  |
| Relationship   | "is-a" (Dog **is an** Animal) | "can-do" (Dog **can** ISwim) |

---

### `const` vs `readonly`

| Aspect         | `const`                      | `readonly`                          |
| -------------- | ---------------------------- | ----------------------------------- |
| When set       | Compile-time                 | Runtime (in constructor or inline)  |
| Always static? | ✅ Yes (implicitly static)   | ❌ No (can be instance or `static`) |
| Allowed types  | Primitives and `string` only | Any type (objects, arrays, etc.)    |
| Changeable?    | Never                        | Only in constructor                 |

```csharp
public class Config
{
    public const double Pi = 3.14159;                  // Compile-time constant
    public const string AppName = "MyApp";

    public readonly string ConnectionString;           // Set once at runtime
    public static readonly DateTime AppStart          // Static readonly
        = DateTime.UtcNow;

    public Config(string connStr)
    {
        ConnectionString = connStr;                    // Allowed in constructor ✅
    }
}
```

---

### Extension Methods

> **Extension methods** allow you to add methods to an existing type **without modifying it** — no inheritance or source access needed.

**Rules:**

- Must be in a **`static` class**
- Method must be **`static`**
- First parameter must have the **`this`** keyword

```csharp
public static class StringExtensions
{
    // Adds IsNullOrEmpty() directly to any string variable
    public static bool IsNullOrEmpty(this string value)
        => string.IsNullOrEmpty(value);

    public static string ToCamelCase(this string value)
        => string.IsNullOrEmpty(value) ? value
           : char.ToLower(value[0]) + value[1..];
}

// Usage — called like an instance method
string name = "Hello";
bool empty = name.IsNullOrEmpty();        // ✅ Extension method call
string camel = "MyVariable".ToCamelCase(); // → "myVariable"
```

> **LINQ is entirely extension methods** on `IEnumerable<T>` and `IQueryable<T>`. `Where`, `Select`, `OrderBy` are all extension methods defined in `System.Linq`.

---

### Reflection

> **Reflection** allows you to **inspect and manipulate types, properties, and methods at runtime** — even when you don't know them at compile time.

```csharp
// Inspect a type at runtime
Type type = typeof(Employee);
Console.WriteLine(type.Name);            // "Employee"

// Get all public properties
PropertyInfo[] props = type.GetProperties();
foreach (var p in props)
    Console.WriteLine($"{p.Name}: {p.PropertyType.Name}");

// Create instance dynamically (without knowing the type at compile time)
object emp = Activator.CreateInstance(typeof(Employee));

// Set a property value dynamically
PropertyInfo nameProp = type.GetProperty("Name");
nameProp.SetValue(emp, "John Doe");

// Invoke a method dynamically
MethodInfo method = type.GetMethod("GetFullName");
var result = method.Invoke(emp, null);
```

**Where Reflection is used:**

| Framework / Tool | How it uses Reflection                      |
| ---------------- | ------------------------------------------- |
| EF Core          | Maps properties to DB columns automatically |
| DI containers    | Resolves constructor parameters at runtime  |
| JSON serializers | Reads/writes properties by name at runtime  |
| Test frameworks  | Discovers `[Test]` methods and runs them    |
| AutoMapper       | Maps property names between objects         |

**Downside:** Slower than direct code (no compile-time checks, uses late binding). Use sparingly in hot paths.

---

## 📝 Quick Recap

| Feature                     | Version | One-liner                                      |
| --------------------------- | ------- | ---------------------------------------------- |
| Primary Constructors        | C# 12   | Define constructor params on class declaration |
| Collection Expressions `[]` | C# 12   | Unified syntax to create any collection        |
| Default Lambda Params       | C# 12   | Lambda can have default values                 |
| Alias Any Type              | C# 12   | `using` for tuples, arrays, generics           |
| Native AOT                  | .NET 8  | Compile to machine code at build time          |
| Frozen Collections          | .NET 8  | Read-only, fast-lookup collections             |
| Minimal APIs                | .NET 6+ | Lightweight API with minimal code              |

---

_[← Back to README](./README.md)_
