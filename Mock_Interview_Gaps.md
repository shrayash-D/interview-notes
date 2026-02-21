# 🚨 Mock Interview Gap Topics — Must Know!

> These topics were **repeatedly asked and failed** in the mock evaluations of your batch (Feb 2026).
> Study these BEFORE your final interview!

---

## 📋 Topics Frequently Failed in Mock Interviews

| # | Topic | Mentioned By |
|---|---|---|
| 1 | Service Lifetime (Transient/Scoped/Singleton) with **examples** | Renuka, Arjun, Amit, Nagul, Shrayash, Sarang |
| 2 | `app.Use` vs `app.Run` in ASP.NET Core | Shrayash, Prashant |
| 3 | `IQueryable` vs `ICollection` vs `IEnumerable` | Arjun, Dipali, Lokesh |
| 4 | Asynchronous Programming (`async`/`await`) | Renuka, Arjun |
| 5 | Reflection in C# | Nitesh |
| 6 | `const` vs `readonly` in C# | Rakshant |
| 7 | Extension Methods in C# | Rakshant |
| 8 | Types of Classes in C# (abstract, sealed, static, partial) | Snehal, Saurabh, Sarang |
| 9 | `[ApiController]` attribute — what does it do? | Prashant |
| 10 | Primary Key vs Unique Key in SQL | Nitesh, Naveen, Nagul |
| 11 | Fluent API in EF Core | Arjun |
| 12 | Role-Based Authorization in ASP.NET Core API | Dhanush, Abhilash, Darshan |
| 13 | Repository Pattern (implementation) | Many candidates |
| 14 | Global Exception Handling in Web API | Dhanush, Siddhant, Devansh, Naveen, Rakshant |
| 15 | CLR in C# | Rakshant |
| 16 | Concurrency Conflicts in EF Core | Prashant |
| 17 | MVC vs Web API | Nirjara |
| 18 | Liskov Substitution Principle (real-time example) | Snehal, Nagul |
| 19 | Window Functions in SQL | Snehal, Lokesh, Rakshant, Sarayu |
| 20 | Types of Functions in SQL (Scalar/Table-Valued) | Nitesh, Saurabh, Sarayu |
| 21 | Cloud Service Model Examples (IaaS/PaaS/SaaS) | Shrayash, Aasawari, Sarayu |
| 22 | Dapper (what is it?) | Lokesh |
| 23 | What is a REST API / stateless? | Sarang, Sarayu |

---

## 🔷 1. Service Lifetime — With Real Examples

> **Most asked topic** — you must explain WITH an example, not just a definition.

| Lifetime | Created | Destroyed | Use When |
|---|---|---|---|
| **Transient** | Every time `.GetService<T>()` is called | After use | Lightweight, stateless (e.g., email formatter) |
| **Scoped** | Once per HTTP request | End of request | Per-request data (e.g., DbContext, current user info) |
| **Singleton** | Once when app starts | App shuts down | Shared global state (e.g., cache, config, logger) |

```csharp
// Registration
builder.Services.AddTransient<IEmailFormatter, EmailFormatter>();   // new instance every time
builder.Services.AddScoped<IEmployeeService, EmployeeService>();    // one per request
builder.Services.AddSingleton<IAppCache, AppCache>();               // one for whole app

// Practical example — why Scoped for DbContext?
// Request 1: User A → Gets DbContext #1 → Tracks User A's changes
// Request 2: User B → Gets DbContext #2 → Tracks User B's changes
// If Singleton → One DbContext shared by all → RACE CONDITION! ❌

// Verify in code
public class DemoController : ControllerBase
{
    private readonly IMyService _s1;
    private readonly IMyService _s2;

    public DemoController(IMyService s1, IMyService s2)
    {
        _s1 = s1;
        _s2 = s2;
    }

    [HttpGet]
    public IActionResult Check()
    {
        // Transient: s1 != s2 (different instances)
        // Scoped:    s1 == s2 (same instance within this request)
        // Singleton: s1 == s2 always, across ALL requests
        return Ok(ReferenceEquals(_s1, _s2));
    }
}
```

**Interview Answer:** "Transient creates a new instance every time it's injected — good for stateless services. Scoped creates one instance per HTTP request — perfect for DbContext because each request should have its own database context. Singleton creates one instance for the entire app lifetime — used for caching or configuration. Using Singleton for DbContext would cause race conditions because multiple requests would share the same change tracker."

---

## 🔷 2. `app.Use` vs `app.Run` vs `app.Map`

> **Frequently missed** — this is about how the middleware pipeline works.

| Method | Calls next? | Terminates pipeline? | Use for |
|---|---|---|---|
| `app.Use()` | ✅ Yes (by calling `next()`) | ❌ No | Middleware that processes and passes along |
| `app.Run()` | ❌ No | ✅ Yes — terminal | Final response, no next middleware |
| `app.Map()` | ✅ Branches | Branches pipeline | Route-specific middleware |

```csharp
// app.Use — processes request AND passes to next
app.Use(async (context, next) =>
{
    Console.WriteLine("Before → Middleware 1");
    await next(context);   // Call next middleware in pipeline
    Console.WriteLine("After ← Middleware 1");
});

app.Use(async (context, next) =>
{
    Console.WriteLine("Before → Middleware 2");
    await next(context);
    Console.WriteLine("After ← Middleware 2");
});

// app.Run — TERMINAL, never calls next
app.Run(async context =>
{
    Console.WriteLine("Terminal middleware — no next!");
    await context.Response.WriteAsync("Final Response");
    // Nothing after this runs!
});

// Output for a request:
// Before → Middleware 1
// Before → Middleware 2
// Terminal middleware — no next!
// After ← Middleware 2
// After ← Middleware 1

// app.Map — branch pipeline for specific path
app.Map("/api", apiApp =>
{
    apiApp.Run(async context =>
        await context.Response.WriteAsync("API branch"));
});
```

**Interview Answer:** "`app.Use` adds middleware that can call `next()` to pass control to the next middleware in the pipeline. `app.Run` is a terminal middleware — it never calls `next()`, so the pipeline stops there. `app.Map` creates a branch for a specific path prefix. If you add `app.Run` before other middleware, those later ones will never execute."

---

## 🔷 3. `IEnumerable` vs `IQueryable` vs `ICollection`

| Interface | Execution | Where runs | LINQ Support | Use Case |
|---|---|---|---|---|
| `IEnumerable<T>` | Immediate (in-memory) | **C# code** (client-side) | In-memory filtering | Iterate in-memory lists, small data |
| `IQueryable<T>` | Deferred (lazy) | **Database** (server-side) | Translated to SQL | EF Core queries — filter/sort in DB |
| `ICollection<T>` | In-memory | C# code | Standard LINQ | Lists with Add/Remove (navigation properties) |

```csharp
// IQueryable — filter runs IN SQL SERVER (efficient)
IQueryable<Employee> query = _context.Employees;  // No SQL yet
query = query.Where(e => e.Salary > 50000);        // Still no SQL
query = query.OrderBy(e => e.Name);                // Still no SQL
var result = query.ToList();  // NOW SQL runs: SELECT * FROM Employees WHERE Salary > 50000 ORDER BY Name

// IEnumerable — LOADS EVERYTHING FIRST, then filters in memory (inefficient!)
IEnumerable<Employee> all = _context.Employees.ToList();  // SELECT * FROM Employees (all rows!)
var filtered = all.Where(e => e.Salary > 50000);           // Filtering in C# (after load)

// Why this matters:
// Table has 1,000,000 employees, 10 have salary > 50000
// IQueryable → fetches 10 rows from DB ✅
// IEnumerable → fetches 1,000,000 rows then filters to 10 ❌ 10x memory, 100x slow

// ICollection — used in navigation properties (EF Core relationships)
public class Department
{
    public ICollection<Employee> Employees { get; set; }  // Loaded into memory
    // NOT IQueryable — because it's already loaded
}

// Summary:
// IEnumerable  — iterate anything (arrays, lists, DB) — LINQ in memory
// IQueryable   — build SQL queries deferred — LINQ translated to SQL
// ICollection  — in-memory collection with Add/Remove/Count
```

**Interview Answer:** "`IQueryable` defers execution and translates LINQ to SQL — filters run in the database. `IEnumerable` loads all data into memory first, then filters in C# — very inefficient for large tables. Use `IQueryable` with EF Core for database queries. `ICollection` is for in-memory collections and is used in navigation properties."

---

## 🔷 4. Asynchronous Programming — `async` / `await`

> **Repeatedly missed** — must understand WHY, not just HOW.

```csharp
// WHY async? — Without async, the thread is BLOCKED waiting for I/O
// With async, the thread is RELEASED and can handle other requests

// Synchronous (bad for web APIs — blocks thread)
public Employee GetEmployee(int id)
{
    Thread.Sleep(2000);  // Thread blocked for 2 seconds ❌
    return _context.Employees.Find(id);
}

// Asynchronous (good — thread freed while waiting)
public async Task<Employee> GetEmployeeAsync(int id)
{
    await Task.Delay(2000);  // Thread released, available for other requests ✅
    return await _context.Employees.FindAsync(id);
}

// Rules:
// 1. Method returns Task or Task<T> (never void, except event handlers)
// 2. Use await keyword to call async methods
// 3. Mark the method as async
// 4. Use Async suffix by convention

// Wrong — causes deadlock in some scenarios
var emp = GetEmployeeAsync(1).Result;    // ❌ .Result blocks synchronously
var emp = GetEmployeeAsync(1).Wait();    // ❌ .Wait() blocks synchronously

// Correct
var emp = await GetEmployeeAsync(1);    // ✅ await properly

// Run multiple async operations in parallel
var empTask   = GetEmployeeAsync(1);
var deptTask  = GetDepartmentAsync(5);

await Task.WhenAll(empTask, deptTask);  // Both run simultaneously
var emp  = await empTask;
var dept = await deptTask;

// async void — ONLY for event handlers (exceptions can't be caught!)
private async void Button_Click(object sender, EventArgs e)  // OK for events
{
    await DoSomethingAsync();
}
```

**Interview Answer:** "Async/await allows non-blocking I/O. When a method hits `await`, the thread is released back to the thread pool to handle other requests — instead of sitting idle waiting for the database/network. The method resumes when the awaited operation completes. For Web APIs, this is critical — a synchronous API can only handle as many concurrent requests as there are threads (typically ~100), but an async API can handle thousands."

---

## 🔷 5. Types of Classes in C#

> **Frequently asked** — know each type with a quick example.

| Class Type | Key Characteristic | Can Instantiate? | Can Inherit? |
|---|---|---|---|
| **Regular class** | Normal class | ✅ Yes | ✅ Yes |
| **Abstract class** | Has `abstract` methods with no body | ❌ No | ✅ Yes (must implement abstract members) |
| **Sealed class** | Cannot be inherited | ✅ Yes | ❌ No |
| **Static class** | Only static members, no instances | ❌ No | ❌ No |
| **Partial class** | Split across multiple files | ✅ Yes | ✅ Yes |

```csharp
// 1. Abstract Class — blueprint, cannot be instantiated
public abstract class Animal
{
    public string Name { get; set; }
    public abstract void MakeSound();  // Must be overridden in child
    public void Sleep() => Console.WriteLine("Sleeping...");  // Can have implementation
}
public class Dog : Animal
{
    public override void MakeSound() => Console.WriteLine("Woof!");
}
// new Animal(); ❌ Error! Cannot instantiate abstract class

// 2. Sealed Class — cannot be inherited
public sealed class Logger
{
    public void Log(string msg) => Console.WriteLine(msg);
}
// public class MyLogger : Logger { } ❌ Error! Cannot inherit from sealed class
// Used to prevent modification (e.g., String class is sealed)

// 3. Static Class — no instances, only shared members
public static class MathHelper
{
    public static int Add(int a, int b) => a + b;
    public static double Pi = 3.14159;
}
MathHelper.Add(1, 2);  // ✅ Called on class, not instance
// new MathHelper(); ❌ Error!

// 4. Partial Class — split across files (used in ASP.NET code-behind)
// File1.cs
public partial class Employee
{
    public string Name { get; set; }
}
// File2.cs
public partial class Employee
{
    public void PrintName() => Console.WriteLine(Name);
}
// Compiler merges them into one class

// 5. Abstract vs Interface — common interview question
// Abstract: shared implementation + state (fields) + "is a" relationship
// Interface: pure contract, no state — "can do" relationship
```

---

## 🔷 6. `const` vs `readonly` in C#

| Feature | `const` | `readonly` |
|---|---|---|
| **Set when?** | Compile time | Runtime (in constructor) |
| **Static?** | Always static | Can be instance or static |
| **Can be object?** | ❌ Only primitives/strings | ✅ Any type |
| **Changed after set?** | ❌ Never | ❌ After constructor |

```csharp
public class AppConfig
{
    // const — value baked in at compile time, always the same
    public const double Pi = 3.14159;
    public const string AppName = "MyApp";

    // readonly — set once in constructor, can differ per instance
    public readonly string ConnectionString;
    public readonly DateTime CreatedAt;

    public AppConfig(string connStr)
    {
        ConnectionString = connStr;   // ✅ Set in constructor
        CreatedAt = DateTime.UtcNow;  // ✅ Different per instance
    }
}

// static readonly — set once for the class (like const but runtime)
public static readonly string Version = GetVersionFromFile();
```

**Interview Answer:** "`const` is a compile-time constant — the value is embedded in compiled code and is always the same. `readonly` is a runtime constant — it can be set in the constructor and can have different values per instance. Use `const` for mathematical constants like Pi. Use `readonly` for values that depend on configuration or runtime data."

---

## 🔷 7. Extension Methods in C#

> Methods that **add new functionality** to existing types without modifying or inheriting them.

```csharp
// Rules:
// 1. Must be in a static class
// 2. Method must be static
// 3. First parameter is the type being extended with 'this' keyword

public static class StringExtensions
{
    // Extends the string type
    public static bool IsNullOrEmpty(this string value)
        => string.IsNullOrEmpty(value);

    public static string ToCamelCase(this string value)
        => char.ToLower(value[0]) + value.Substring(1);

    public static string Truncate(this string value, int maxLength)
        => value.Length <= maxLength ? value : value.Substring(0, maxLength) + "...";
}

// Usage — looks like it's a method ON string
string name = "Hello World";
bool isEmpty = name.IsNullOrEmpty();    // ✅ Extension method call
string camel = name.ToCamelCase();     // ✅ helloWorld
string short_ = name.Truncate(5);     // ✅ Hello...

// Real-world use: extending IQueryable for reusable filters
public static class QueryExtensions
{
    public static IQueryable<Employee> ActiveOnly(this IQueryable<Employee> query)
        => query.Where(e => e.IsActive);

    public static IQueryable<Employee> InDepartment(this IQueryable<Employee> query, int deptId)
        => query.Where(e => e.DepartmentId == deptId);
}

// Usage
var active = _context.Employees.ActiveOnly().InDepartment(5).ToList();
```

**Interview Answer:** "Extension methods let you add new methods to existing types without modifying them or using inheritance. They're defined as static methods in a static class, with the first parameter prefixed by `this` to specify which type is being extended. LINQ itself is built entirely with extension methods on `IEnumerable<T>` and `IQueryable<T>`."

---

## 🔷 8. Reflection in C#

> **Reflection** allows you to inspect and manipulate types, methods, and properties at **runtime** — even without knowing them at compile time.

```csharp
// Get type info at runtime
Type type = typeof(Employee);
Console.WriteLine(type.Name);          // "Employee"
Console.WriteLine(type.FullName);      // "MyApp.Models.Employee"
Console.WriteLine(type.IsClass);       // true

// Get all properties
PropertyInfo[] props = type.GetProperties();
foreach (var prop in props)
    Console.WriteLine($"{prop.Name}: {prop.PropertyType.Name}");

// Get all methods
MethodInfo[] methods = type.GetMethods();

// Create instance at runtime
object emp = Activator.CreateInstance(typeof(Employee));

// Set/Get property value at runtime
var nameProperty = type.GetProperty("Name");
nameProperty.SetValue(emp, "John");
var value = nameProperty.GetValue(emp);  // "John"

// Call method at runtime
var method = type.GetMethod("GetFullName");
method.Invoke(emp, null);

// Common uses of Reflection:
// - ORMs (EF Core maps class properties to DB columns)
// - Dependency Injection (resolves types at runtime)
// - Unit test frameworks (find [Test] methods)
// - Serialization (JSON serializers read property names)
// - Attribute reading
var attributes = type.GetCustomAttributes(typeof(TableAttribute), false);
```

**Interview Answer:** "Reflection allows you to inspect type metadata and manipulate objects at runtime. You can get property names, call methods, and create instances without knowing the type at compile time. It's used by EF Core to map properties to database columns, by DI containers to resolve services, and by serializers like `System.Text.Json`. The downside is it's slower than direct calls and bypasses compile-time safety."

---

## 🔷 9. CLR — Common Language Runtime

```
Source Code (.cs)
    ↓ C# Compiler
MSIL / CIL (Intermediate Language — .dll/.exe)
    ↓ CLR (JIT Compiler)
Native Machine Code (runs on CPU)
```

| CLR Feature | What it does |
|---|---|
| **JIT Compilation** | Converts IL to native CPU code at runtime |
| **Garbage Collection** | Automatically frees unused memory |
| **Type Safety** | Prevents invalid memory access |
| **Exception Handling** | try/catch/finally infrastructure |
| **Security** | Code Access Security, type verification |
| **Thread Management** | Thread pool management |

**Interview Answer:** "The CLR is the execution engine for .NET. C# code is first compiled to Intermediate Language (IL), then at runtime the CLR's JIT compiler converts IL to native machine code. The CLR also manages memory via garbage collection, handles exceptions, enforces type safety, and manages threads."

---

## 🔷 10. `[ApiController]` Attribute — What Does It Do?

> Without `[ApiController]`, a Web API controller behaves like an MVC controller. With it, you get 4 automatic behaviors.

```csharp
[ApiController]       // ← This attribute
[Route("api/[controller]")]
public class EmployeesController : ControllerBase { }
```

**What `[ApiController]` enables automatically:**

```csharp
// 1. AUTOMATIC MODEL VALIDATION
// Without [ApiController] — you must check ModelState manually
public IActionResult Create(Employee emp)
{
    if (!ModelState.IsValid) return BadRequest(ModelState);  // Must write this ❌
    ...
}

// With [ApiController] — returns 400 automatically if model is invalid
public IActionResult Create(Employee emp)
{
    // No need to check ModelState! ✅ Done automatically
    ...
}

// 2. AUTOMATIC [FromBody] FOR COMPLEX TYPES
// Without [ApiController]
public IActionResult Create([FromBody] Employee emp) { ... }  // Must specify [FromBody]

// With [ApiController]
public IActionResult Create(Employee emp) { ... }  // [FromBody] inferred automatically ✅

// 3. PROBLEM DETAILS FOR ERRORS
// Returns standardized RFC 7807 error format:
// { "type": "...", "title": "Bad Request", "status": 400, "errors": { "Name": ["required"] } }

// 4. ATTRIBUTE ROUTING REQUIRED
// Controllers with [ApiController] MUST use [Route] or [HttpGet]/[HttpPost] etc.
// Convention routing (like MVC's {controller}/{action}) doesn't work.
```

**Interview Answer:** "`[ApiController]` enables 4 automatic behaviors: automatic model validation returning 400 if invalid, automatic `[FromBody]` binding for complex types, standardized RFC 7807 error responses, and enforces attribute routing. Without it, you'd need to manually call `ModelState.IsValid` in every action."

---

## 🔷 11. Primary Key vs Unique Key in SQL

| Feature | Primary Key | Unique Key |
|---|---|---|
| **NULL allowed?** | ❌ No (NOT NULL enforced) | ✅ Yes (one NULL allowed) |
| **Per table** | Only **ONE** primary key | **Multiple** unique keys |
| **Creates index?** | Clustered index (by default) | Non-clustered index |
| **Purpose** | Uniquely identify the row | Enforce uniqueness on non-PK columns |

```sql
CREATE TABLE Employees (
    EmployeeId   INT PRIMARY KEY,      -- Primary Key: NOT NULL, clustered index
    Email        VARCHAR(100) UNIQUE,  -- Unique: allows NULL, non-clustered index
    Phone        VARCHAR(20)  UNIQUE,  -- Multiple unique keys allowed
    Name         VARCHAR(100) NOT NULL
);

-- Composite Primary Key
CREATE TABLE OrderItems (
    OrderId    INT,
    ProductId  INT,
    PRIMARY KEY (OrderId, ProductId)   -- Composite PK
);

-- Foreign Key references Primary Key
ALTER TABLE Orders
ADD CONSTRAINT FK_Employee FOREIGN KEY (EmployeeId) REFERENCES Employees(EmployeeId);
```

**Interview Answer:** "Primary Key uniquely identifies each row — it cannot be NULL and there can only be one per table. It creates a clustered index. Unique Key also enforces uniqueness but allows one NULL value and you can have multiple unique keys per table. It creates a non-clustered index. A Foreign Key references a Primary Key in another table."

---

## 🔷 12. Fluent API in EF Core

> **Fluent API** is an alternative to Data Annotations — used in `OnModelCreating()` to configure entities with a method-chaining syntax. **More powerful than annotations.**

```csharp
public class AppDbContext : DbContext
{
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Employee>(entity =>
        {
            // Table and column names
            entity.ToTable("tbl_Employees");
            entity.Property(e => e.Name).HasColumnName("FullName");

            // Required and max length
            entity.Property(e => e.Name).IsRequired().HasMaxLength(100);

            // Column type
            entity.Property(e => e.Salary).HasColumnType("decimal(18,2)");

            // Default value
            entity.Property(e => e.CreatedAt).HasDefaultValueSql("GETUTCDATE()");

            // Unique constraint
            entity.HasIndex(e => e.Email).IsUnique();

            // Relationships
            entity.HasOne(e => e.Department)
                  .WithMany(d => d.Employees)
                  .HasForeignKey(e => e.DepartmentId)
                  .OnDelete(DeleteBehavior.Restrict);  // No cascade delete
        });
    }
}

// Data Annotations vs Fluent API
// Annotation: [Required], [MaxLength(100)], [Column("FullName")]
// Fluent API: .IsRequired().HasMaxLength(100).HasColumnName("FullName")

// Use Fluent API when:
// - You can't modify the entity class (e.g., external library)
// - Need complex configurations (composite keys, table splitting)
// - Want to keep entity classes clean (no attributes)
```

**Interview Answer:** "Fluent API configures entity-database mapping in `OnModelCreating()` using method chaining. It's more powerful than Data Annotations — you can configure things like composite keys, table splitting, cascade behavior, and default values that annotations can't do. The entity classes stay clean with no attributes."

---

## 🔷 13. Role-Based Authorization in ASP.NET Core Web API

```csharp
// Program.cs — Add Authorization with policies
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("AdminOnly", policy => policy.RequireRole("Admin"));
    options.AddPolicy("ManagerOrAdmin", policy => policy.RequireRole("Manager", "Admin"));
    options.AddPolicy("AtLeast18", policy => policy.RequireClaim("Age", "18", "19", "20"));
});

// In JWT token — include roles in claims
var claims = new[]
{
    new Claim(ClaimTypes.Name, "john"),
    new Claim(ClaimTypes.Role, "Admin"),       // Role claim
    new Claim(ClaimTypes.Role, "Manager"),     // Multiple roles
};

// Controller — protect with roles
[ApiController]
[Route("api/[controller]")]
[Authorize]                                    // Must be logged in
public class EmployeesController : ControllerBase
{
    [HttpGet]
    [Authorize(Roles = "Admin,Manager")]       // Admin OR Manager
    public IActionResult GetAll() { ... }

    [HttpDelete("{id}")]
    [Authorize(Roles = "Admin")]               // Admin only
    public IActionResult Delete(int id) { ... }

    [HttpGet("public")]
    [AllowAnonymous]                           // No auth needed
    public IActionResult GetPublic() { ... }

    [HttpGet("policy")]
    [Authorize(Policy = "AdminOnly")]          // Using named policy
    public IActionResult GetAdmin() { ... }
}

// Access current user in controller
var userId = User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
var username = User.Identity.Name;
bool isAdmin = User.IsInRole("Admin");
```

**Interview Answer:** "Role-based authorization in Web API uses `[Authorize(Roles = 'Admin')]` on controllers or actions. The user's roles are stored as claims in the JWT token. The middleware validates the token and checks if the role claim matches what the endpoint requires. You can also define named policies for complex authorization logic."

---

## 🔷 14. MVC vs Web API

| Feature | MVC | Web API |
|---|---|---|
| **Returns** | HTML Views (Razor) | JSON/XML data |
| **Base class** | `Controller` | `ControllerBase` |
| **URL** | `/employees/create` | `/api/employees` |
| **Used for** | Web applications (browser) | Mobile apps, SPAs, microservices |
| **View engine** | Razor (.cshtml) | None |
| **Model binding** | Forms, query strings | JSON body, query strings |

```csharp
// MVC Controller — returns Views (HTML)
public class EmployeesController : Controller  // ← Controller (has View support)
{
    public IActionResult Index()
    {
        var employees = _context.Employees.ToList();
        return View(employees);  // Renders .cshtml file
    }
}

// Web API Controller — returns Data (JSON)
[ApiController]
[Route("api/[controller]")]
public class EmployeesController : ControllerBase  // ← ControllerBase (no Views)
{
    [HttpGet]
    public async Task<ActionResult<IEnumerable<Employee>>> GetAll()
    {
        return Ok(await _context.Employees.ToListAsync());  // Returns JSON
    }
}

// In ASP.NET Core 8 — you can have BOTH in the same app!
// MVC for web pages + Web API for AJAX/mobile calls
```

---

## 🔷 15. Window Functions in SQL — Quick Recap

> Since many candidates failed this, here's a fast review.

```sql
-- ROW_NUMBER — unique sequential number
SELECT Name, Salary,
       ROW_NUMBER() OVER (PARTITION BY DeptId ORDER BY Salary DESC) AS RowNum
FROM Employees;

-- RANK — same rank for ties, skips numbers (1,1,3)
SELECT Name, Salary,
       RANK() OVER (ORDER BY Salary DESC) AS Rank
FROM Employees;

-- DENSE_RANK — same rank for ties, NO skip (1,1,2)
SELECT Name, Salary,
       DENSE_RANK() OVER (ORDER BY Salary DESC) AS DenseRank
FROM Employees;

-- Interview question: Get top 2 salaries per department
SELECT * FROM (
    SELECT *, DENSE_RANK() OVER (PARTITION BY DeptId ORDER BY Salary DESC) AS dr
    FROM Employees
) t
WHERE dr <= 2;

-- LEAD / LAG — access next/previous row
SELECT Name, Salary,
       LAG(Salary) OVER (ORDER BY Salary)  AS PrevSalary,   -- Previous row
       LEAD(Salary) OVER (ORDER BY Salary) AS NextSalary    -- Next row
FROM Employees;

-- RANK vs DENSE_RANK (exact difference)
Salaries: 90000, 85000, 85000, 70000
RANK:       1      2      2      4    ← Skips 3
DENSE_RANK: 1      2      2      3    ← No skip
```

---

## 🔷 16. Types of SQL Functions

```sql
-- 1. Scalar Function — returns ONE value
CREATE FUNCTION dbo.GetFullName (@FirstName NVARCHAR(50), @LastName NVARCHAR(50))
RETURNS NVARCHAR(100)
AS
BEGIN
    RETURN @FirstName + ' ' + @LastName
END

-- Usage
SELECT dbo.GetFullName(FirstName, LastName) AS FullName FROM Employees;

-- 2. Inline Table-Valued Function — returns a TABLE (single SELECT)
CREATE FUNCTION dbo.GetEmployeesByDept (@DeptId INT)
RETURNS TABLE
AS
RETURN (
    SELECT * FROM Employees WHERE DepartmentId = @DeptId
);

-- Usage
SELECT * FROM dbo.GetEmployeesByDept(3);

-- 3. Multi-Statement Table-Valued Function — returns a TABLE with logic
CREATE FUNCTION dbo.GetTopEarners (@TopN INT)
RETURNS @Result TABLE (Name NVARCHAR(100), Salary DECIMAL(18,2))
AS
BEGIN
    INSERT INTO @Result
    SELECT TOP (@TopN) Name, Salary FROM Employees ORDER BY Salary DESC;
    RETURN;
END

-- Functions vs Stored Procedures
-- Function: returns a value/table, used in SELECT, no DML (no INSERT/UPDATE/DELETE)
-- SP: no return in SELECT, can do anything including DML
```

---

## 🔷 17. IaaS, PaaS, SaaS — Real Examples

```
IaaS (Infrastructure as a Service) — You manage OS + App
  Azure VMs        → You install Windows/Linux, IIS, your app
  AWS EC2          → Same
  Example: "We use Azure VMs to host our SQL Server because we need full control over configuration."

PaaS (Platform as a Service) — You manage only the App + Data
  Azure App Service → Upload your .NET app, Azure manages the OS, runtime, scaling
  Azure SQL Database → Just use SQL — no server management
  Example: "We deploy to Azure App Service — we just push code, Azure handles patching and scaling."

SaaS (Software as a Service) — You just USE it
  Microsoft 365     → Word, Excel — just log in and use
  Salesforce        → CRM — no installation
  GitHub            → Source control — just use it
  Example: "We use Azure DevOps for CI/CD — it's SaaS, no setup needed."

Interview Answer: "IaaS gives you virtual machines — you manage OS and above.
PaaS gives you a managed platform — you manage only your app and data.
SaaS gives you ready-to-use software — you manage nothing."
```

---

## 🔷 18. `what is Dapper` (Full Explanation)

> Already covered in `Module4_EFCore_Dapper.md` but here's the quick interview answer.

**Interview Answer:** "Dapper is a micro-ORM (Object Relational Mapper) for .NET. Unlike EF Core which auto-generates SQL, Dapper lets you write raw SQL and maps the results to C# objects automatically. It's extremely fast — nearly as fast as raw ADO.NET — because there's no overhead from change tracking or query translation. I use EF Core for standard CRUD operations and Dapper for complex queries or performance-critical reports."

---

## 📊 Coverage Summary from Mock Interviews

| Topic | In Notes? | File |
|---|---|---|
| Service Lifetime with examples | ✅ Covered + expanded here | Module5, this file |
| `app.Use` vs `app.Run` | ✅ Expanded here | This file, Module5 |
| IEnumerable vs IQueryable vs ICollection | ✅ Detailed here | This file |
| async/await | ✅ Detailed here | This file |
| Types of Classes (abstract/sealed/static/partial) | ✅ Added here | This file |
| const vs readonly | ✅ Added here | This file |
| Extension Methods | ✅ Added here | This file |
| Reflection | ✅ Added here | This file |
| CLR | ✅ Added here | This file |
| `[ApiController]` attribute | ✅ Added here | This file |
| Primary Key vs Unique Key | ✅ Added here | This file |
| Fluent API in EF Core | ✅ Added here | This file |
| Role-Based Authorization | ✅ Added here | This file, Module6 |
| MVC vs Web API | ✅ Added here | This file |
| Window Functions (recap) | ✅ Module3 + recap here | Module3, this file |
| Types of SQL Functions | ✅ Added here | Module3, this file |
| IaaS/PaaS/SaaS examples | ✅ Added here | Module10, this file |
| Repository Pattern | ✅ Bonus_Extra_Topics.md | Bonus file |
| Global Exception Handling | ✅ Module6 | Module6 |
| Dapper | ✅ Module4 | Module4 |
| AutoMapper | ✅ Bonus file | Bonus file |

---

*[← Back to README](./README.md)*
