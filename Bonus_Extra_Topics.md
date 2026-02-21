# 📌 BONUS MODULE: Extra Important Interview Topics

> These topics are NOT in the official syllabus but are **very commonly asked** in .NET Full Stack interviews.
> Study these after finishing all 5 modules!

---

## 🔷 1. DTO (Data Transfer Object)

### What is a DTO?
> A **DTO** is a simple C# class used to **transfer data between layers** (e.g., from database to API response). It only contains the fields you want to expose — no business logic.

**Why use DTOs?**
- Avoids exposing your database model directly to the outside world
- Hides sensitive fields (e.g., `PasswordHash`, `InternalId`)
- Shapes data exactly how the consumer needs it
- Prevents over-posting attacks

```csharp
// ❌ Bad — returning the Entity directly exposes all DB fields
public Employee GetEmployee(int id) => _context.Employees.Find(id);

// ✅ Good — return only what the caller needs via DTO
public class EmployeeDto
{
    public string FullName { get; set; }
    public string Email { get; set; }
    public string DepartmentName { get; set; }
    // No PasswordHash, no internal IDs ✅
}

// Map Entity → DTO manually
public EmployeeDto GetEmployee(int id)
{
    var emp = _context.Employees.Include(e => e.Department).FirstOrDefault(e => e.Id == id);
    return new EmployeeDto
    {
        FullName = emp.Name,
        Email = emp.Email,
        DepartmentName = emp.Department.Name
    };
}
```

**Interview Answer:** "A DTO is a plain class used to transfer data between layers or over the network. It helps decouple the database model from what's exposed to the API client, and lets you shape responses exactly how you need them without exposing sensitive or irrelevant data."

---

## 🔷 2. AutoMapper

### What is AutoMapper?
> **AutoMapper** is a library that **automatically maps** properties from one object to another (e.g., `Employee` → `EmployeeDto`), eliminating manual mapping code.

```bash
dotnet add package AutoMapper
dotnet add package AutoMapper.Extensions.Microsoft.DependencyInjection
```

```csharp
// 1. Create a Mapping Profile
public class MappingProfile : Profile
{
    public MappingProfile()
    {
        CreateMap<Employee, EmployeeDto>()
            .ForMember(dest => dest.FullName, opt => opt.MapFrom(src => src.Name))
            .ForMember(dest => dest.DepartmentName, opt => opt.MapFrom(src => src.Department.Name));

        CreateMap<CreateEmployeeDto, Employee>(); // Reverse map for input
    }
}

// 2. Register in Program.cs
builder.Services.AddAutoMapper(typeof(MappingProfile));

// 3. Use in Controller/Service
public class EmployeeController : Controller
{
    private readonly IMapper _mapper;

    public EmployeeController(IMapper mapper) { _mapper = mapper; }

    public IActionResult Get(int id)
    {
        var emp = _context.Employees.Find(id);
        var dto = _mapper.Map<EmployeeDto>(emp);  // ✅ Auto-mapped!
        return Ok(dto);
    }
}
```

**Interview Answer:** "AutoMapper is a library that automatically maps properties between objects using conventions or explicit configuration. It reduces boilerplate mapping code when converting entities to DTOs and vice versa."

---

## 🔷 3. Repository Pattern

### What is the Repository Pattern?
> The **Repository Pattern** is a design pattern that **abstracts the data access layer**. Instead of writing EF Core/Dapper queries directly in controllers, you put them in a "repository" class behind an interface.

**Why?**
- Controllers stay clean (only business logic)
- Easy to unit test (mock the repository)
- Easy to swap data access technology (e.g., switch from EF Core to Dapper)
- Follows SRP and DIP (SOLID principles)

```csharp
// 1. Define the Interface (contract)
public interface IEmployeeRepository
{
    Task<IEnumerable<Employee>> GetAllAsync();
    Task<Employee> GetByIdAsync(int id);
    Task AddAsync(Employee employee);
    Task UpdateAsync(Employee employee);
    Task DeleteAsync(int id);
}

// 2. Implement the Interface
public class EmployeeRepository : IEmployeeRepository
{
    private readonly AppDbContext _context;

    public EmployeeRepository(AppDbContext context)
    {
        _context = context;
    }

    public async Task<IEnumerable<Employee>> GetAllAsync()
        => await _context.Employees.AsNoTracking().ToListAsync();

    public async Task<Employee> GetByIdAsync(int id)
        => await _context.Employees.FindAsync(id);

    public async Task AddAsync(Employee employee)
    {
        await _context.Employees.AddAsync(employee);
        await _context.SaveChangesAsync();
    }

    public async Task UpdateAsync(Employee employee)
    {
        _context.Employees.Update(employee);
        await _context.SaveChangesAsync();
    }

    public async Task DeleteAsync(int id)
    {
        var emp = await _context.Employees.FindAsync(id);
        if (emp != null)
        {
            _context.Employees.Remove(emp);
            await _context.SaveChangesAsync();
        }
    }
}

// 3. Register in Program.cs
builder.Services.AddScoped<IEmployeeRepository, EmployeeRepository>();

// 4. Inject into Controller — Controller never touches DbContext directly!
public class EmployeeController : Controller
{
    private readonly IEmployeeRepository _repo;

    public EmployeeController(IEmployeeRepository repo)
    {
        _repo = repo;
    }

    public async Task<IActionResult> Index()
    {
        var employees = await _repo.GetAllAsync();
        return View(employees);
    }
}
```

**Interview Answer:** "The Repository Pattern abstracts the data access logic behind an interface. The controller calls the repository interface, and the implementation handles EF Core or Dapper queries. This makes the code testable (you can mock the interface), follows SOLID principles, and makes it easy to swap databases."

---

## 🔷 4. Generic Repository Pattern

> A **Generic Repository** avoids writing a separate repository for every entity — one class handles all entities using generics.

```csharp
// Generic Interface
public interface IRepository<T> where T : class
{
    Task<IEnumerable<T>> GetAllAsync();
    Task<T> GetByIdAsync(int id);
    Task AddAsync(T entity);
    Task UpdateAsync(T entity);
    Task DeleteAsync(int id);
}

// Generic Implementation
public class Repository<T> : IRepository<T> where T : class
{
    private readonly AppDbContext _context;
    private readonly DbSet<T> _dbSet;

    public Repository(AppDbContext context)
    {
        _context = context;
        _dbSet = context.Set<T>();  // Gets the correct DbSet for T
    }

    public async Task<IEnumerable<T>> GetAllAsync()
        => await _dbSet.AsNoTracking().ToListAsync();

    public async Task<T> GetByIdAsync(int id)
        => await _dbSet.FindAsync(id);

    public async Task AddAsync(T entity)
    {
        await _dbSet.AddAsync(entity);
        await _context.SaveChangesAsync();
    }

    public async Task UpdateAsync(T entity)
    {
        _dbSet.Update(entity);
        await _context.SaveChangesAsync();
    }

    public async Task DeleteAsync(int id)
    {
        var entity = await _dbSet.FindAsync(id);
        if (entity != null)
        {
            _dbSet.Remove(entity);
            await _context.SaveChangesAsync();
        }
    }
}

// Register — one line covers ALL entities
builder.Services.AddScoped(typeof(IRepository<>), typeof(Repository<>));

// Usage
private readonly IRepository<Employee> _employeeRepo;
private readonly IRepository<Department> _deptRepo;
```

---

## 🔷 5. Unit of Work Pattern

### What is Unit of Work?
> **Unit of Work** keeps track of multiple repository operations and **commits them all in one transaction**. Instead of each repository calling `SaveChanges()` independently, the Unit of Work does it once.

**Why?**
- Ensures atomicity across multiple repositories
- Avoids partial saves (e.g., employee saved but department not)
- Single `SaveChanges()` = better performance

```csharp
// 1. Define Unit of Work Interface
public interface IUnitOfWork : IDisposable
{
    IRepository<Employee> Employees { get; }
    IRepository<Department> Departments { get; }
    Task<int> SaveChangesAsync();
}

// 2. Implement it
public class UnitOfWork : IUnitOfWork
{
    private readonly AppDbContext _context;

    public UnitOfWork(AppDbContext context)
    {
        _context = context;
        Employees = new Repository<Employee>(context);
        Departments = new Repository<Department>(context);
    }

    public IRepository<Employee> Employees { get; private set; }
    public IRepository<Department> Departments { get; private set; }

    public async Task<int> SaveChangesAsync()
        => await _context.SaveChangesAsync();

    public void Dispose() => _context.Dispose();
}

// 3. Register
builder.Services.AddScoped<IUnitOfWork, UnitOfWork>();

// 4. Usage — multiple operations in one transaction
public class OrderService
{
    private readonly IUnitOfWork _uow;

    public OrderService(IUnitOfWork uow) { _uow = uow; }

    public async Task CreateEmployeeWithDept(Employee emp, Department dept)
    {
        await _uow.Departments.AddAsync(dept);
        emp.DepartmentId = dept.DepartmentId;
        await _uow.Employees.AddAsync(emp);

        await _uow.SaveChangesAsync();  // ONE commit for both ✅
    }
}
```

**Interview Answer:** "Unit of Work wraps multiple repository operations and commits them as a single transaction. It avoids partial saves and improves performance by calling SaveChanges() just once. It's commonly used together with the Repository Pattern."

---

## 🔷 6. Middleware in ASP.NET Core (Deep Dive)

### What is Middleware?
> **Middleware** components form a **pipeline** that processes HTTP requests and responses. Each component can:
> - Handle the request itself
> - Pass it to the next component (`await next()`)
> - Or do both (before AND after)

```csharp
// Custom Middleware — Log request time
public class RequestLoggingMiddleware
{
    private readonly RequestDelegate _next;

    public RequestLoggingMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        var start = DateTime.UtcNow;

        Console.WriteLine($"→ {context.Request.Method} {context.Request.Path}");

        await _next(context);  // Call next middleware

        var elapsed = DateTime.UtcNow - start;
        Console.WriteLine($"← Response: {context.Response.StatusCode} in {elapsed.TotalMilliseconds}ms");
    }
}

// Register in Program.cs
app.UseMiddleware<RequestLoggingMiddleware>();

// OR inline middleware
app.Use(async (context, next) =>
{
    // Before
    await next();
    // After
});

// app.Run() — Terminal middleware (no next)
app.Run(async context =>
{
    await context.Response.WriteAsync("Final response!");
});
```

### Middleware Order (Critical!)
```
Request ──────────────────────────────────────────────────► 
   ↓ UseExceptionHandler
   ↓ UseHsts
   ↓ UseHttpsRedirection
   ↓ UseStaticFiles
   ↓ UseRouting
   ↓ UseAuthentication   ← Must be before Authorization
   ↓ UseAuthorization
   ↓ UseSession
   ↓ MapControllers
Response ◄────────────────────────────────────────────────── 
```

**Interview Answer:** "Middleware is a pipeline of components that handle HTTP requests and responses in order. Each middleware can process the request, call the next one, and process the response on the way back. Order matters — for example, Authentication must always come before Authorization."

---

## 🔷 7. Authentication & Authorization in ASP.NET Core

### Authentication vs Authorization
| Concept | Question it answers | Example |
|---|---|---|
| **Authentication** | **Who are you?** | Login with username/password → JWT token issued |
| **Authorization** | **What can you do?** | Only Admins can access `/admin` |

### JWT (JSON Web Token) Authentication

```csharp
// Program.cs — Add JWT Auth
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = "yourApp",
            ValidAudience = "yourApp",
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes("YourSuperSecretKey123!"))
        };
    });

app.UseAuthentication();  // Must come before UseAuthorization
app.UseAuthorization();

// Generate JWT Token
public string GenerateToken(string username)
{
    var claims = new[]
    {
        new Claim(ClaimTypes.Name, username),
        new Claim(ClaimTypes.Role, "Admin")
    };

    var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes("YourSuperSecretKey123!"));
    var creds = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);

    var token = new JwtSecurityToken(
        issuer: "yourApp",
        audience: "yourApp",
        claims: claims,
        expires: DateTime.UtcNow.AddHours(1),
        signingCredentials: creds
    );

    return new JwtSecurityTokenHandler().WriteToken(token);
}

// Protect endpoints
[Authorize]                    // Any authenticated user
public IActionResult Profile() { ... }

[Authorize(Roles = "Admin")]   // Only Admins
public IActionResult AdminDashboard() { ... }

[AllowAnonymous]               // Everyone (even unauthenticated)
public IActionResult Login() { ... }
```

**Interview Answer:** "Authentication verifies who the user is (login). Authorization checks what they're allowed to do (permissions). In ASP.NET Core, JWT Bearer tokens are commonly used — the client sends a token in the `Authorization` header, and the middleware validates it before the request reaches the controller."

---

## 🔷 8. Service Layer Pattern

### What is a Service Layer?
> A **Service Layer** sits between the Controller and the Repository. It contains **business logic** so the controller stays thin.

```
Controller → Service → Repository → Database
```

```csharp
// Without Service Layer (bad — business logic in Controller)
public async Task<IActionResult> Create(Employee emp)
{
    if (emp.Salary < 0) return BadRequest("Salary cannot be negative"); // ❌ Logic in Controller
    if (await _repo.EmailExistsAsync(emp.Email)) return BadRequest("Email taken"); // ❌
    await _repo.AddAsync(emp);
    await _emailService.SendWelcomeEmailAsync(emp.Email); // ❌
    return Ok();
}

// With Service Layer (good — controller just delegates)
public interface IEmployeeService
{
    Task<Result> CreateEmployeeAsync(CreateEmployeeDto dto);
}

public class EmployeeService : IEmployeeService
{
    private readonly IRepository<Employee> _repo;
    private readonly IEmailService _emailService;

    public EmployeeService(IRepository<Employee> repo, IEmailService emailService)
    {
        _repo = repo;
        _emailService = emailService;
    }

    public async Task<Result> CreateEmployeeAsync(CreateEmployeeDto dto)
    {
        if (dto.Salary < 0) return Result.Fail("Salary cannot be negative");
        if (await _repo.EmailExistsAsync(dto.Email)) return Result.Fail("Email taken");

        var employee = new Employee { Name = dto.Name, Email = dto.Email, Salary = dto.Salary };
        await _repo.AddAsync(employee);
        await _emailService.SendWelcomeEmailAsync(dto.Email);

        return Result.Ok();
    }
}

// Controller is now just 3 lines!
public async Task<IActionResult> Create(CreateEmployeeDto dto)
{
    var result = await _employeeService.CreateEmployeeAsync(dto);
    return result.IsSuccess ? Ok() : BadRequest(result.Error);
}
```

**Interview Answer:** "The Service Layer holds the business logic of the application. The Controller just receives the HTTP request and delegates to the service. The service calls the repository for data. This keeps controllers thin, logic testable, and code organized."

---

## 🔷 9. Circular Reference in EF Core / JSON Serialization

### The Problem
> When you serialize an `Employee` that has a `Department`, and `Department` has a list of `Employees`, JSON serializer goes into an **infinite loop**.

```csharp
// Problem: Employee → Department → Employees → Employee → ... 💥
public class Employee
{
    public Department Department { get; set; }
}
public class Department
{
    public ICollection<Employee> Employees { get; set; }
}
```

### Solutions

```csharp
// Option 1: Use DTOs (Best approach — never serialize entities directly!)
var dto = new EmployeeDto { Name = emp.Name, DeptName = emp.Department.Name };
return Ok(dto);

// Option 2: Configure JSON to handle cycles (ASP.NET Core 8)
builder.Services.AddControllers().AddJsonOptions(options =>
{
    options.JsonSerializerOptions.ReferenceHandler = ReferenceHandler.IgnoreCycles;
});

// Option 3: [JsonIgnore] on one side of the relationship
public class Department
{
    public int DepartmentId { get; set; }
    public string Name { get; set; }

    [JsonIgnore]  // Break the cycle
    public ICollection<Employee> Employees { get; set; }
}
```

**Interview Answer:** "Circular reference happens when two entities reference each other — serializing one causes an infinite loop. The best fix is to use DTOs and never serialize EF Core entities directly. Alternatively, use `ReferenceHandler.IgnoreCycles` in the JSON serializer options."

---

## 🔷 10. Locking and Blocking in SQL Server

### Types of Locks
| Lock | Description |
|---|---|
| **Shared (S)** | Read lock — multiple readers allowed at the same time |
| **Exclusive (X)** | Write lock — only one writer, blocks all readers |
| **Update (U)** | Intent to update — prevents deadlocks when upgrading from S to X |
| **Row-Level Lock** | Locks a single row |
| **Page-Level Lock** | Locks a page of data (8KB) |
| **Table-Level Lock** | Locks the entire table |

### Blocking
> **Blocking** happens when one transaction **holds a lock** and another transaction is **waiting** for it.

```sql
-- Check current blocking
SELECT
    blocking_session_id AS BlockingBy,
    session_id AS BlockedSession,
    wait_type,
    wait_time / 1000 AS WaitSeconds,
    sql_text = (SELECT text FROM sys.dm_exec_sql_text(sql_handle))
FROM sys.dm_exec_requests
WHERE blocking_session_id > 0;
```

### Preventing Deadlocks
1. **Always access tables in the same order** across transactions
2. **Keep transactions short** — commit as soon as possible
3. **Use proper indexes** — reduces lock contention
4. **Use `READ COMMITTED SNAPSHOT`** isolation — readers don't block writers
5. **Use `NOLOCK` hint** for read-only queries where dirty reads are acceptable

```sql
-- NOLOCK hint (dirty reads — use with caution)
SELECT * FROM Employees WITH (NOLOCK);

-- Enable RCSI (Read Committed Snapshot Isolation)
ALTER DATABASE MyDb SET READ_COMMITTED_SNAPSHOT ON;
```

---

## 🔷 11. Nested Transactions in SQL Server

> SQL Server **does not truly support nested transactions**. The inner `COMMIT` does nothing — only the outermost `COMMIT` actually commits. But inner `ROLLBACK` rolls back **everything**.

```sql
BEGIN TRANSACTION;      -- @@TRANCOUNT = 1
    INSERT INTO Orders VALUES (1, 'Order1');

    BEGIN TRANSACTION;  -- @@TRANCOUNT = 2 (just increments counter)
        INSERT INTO Orders VALUES (2, 'Order2');
    COMMIT;             -- @@TRANCOUNT = 1 (does NOT commit yet!)

COMMIT;                 -- @@TRANCOUNT = 0 (BOTH inserts committed now)

-- ROLLBACK in inner transaction = rolls back EVERYTHING
BEGIN TRANSACTION;
    INSERT INTO Orders VALUES (1, 'Order1');
    BEGIN TRANSACTION;
        INSERT INTO Orders VALUES (2, 'Order2');
    ROLLBACK;           -- ⚠️ Rolls back ALL transactions!
```

**Interview Answer:** "SQL Server tracks nested transactions using `@@TRANCOUNT`. Each `BEGIN TRANSACTION` increments it, each `COMMIT` decrements it. Only when `@@TRANCOUNT` reaches 0 does the data actually get committed. A `ROLLBACK` in any nested level rolls back everything to the outermost transaction."

---

## 🔷 12. Updatable Views in SQL Server

### Rules for Updating a View
A view is updatable if:
- ✅ It's based on a **single table**
- ✅ It contains the table's **Primary Key**
- ✅ No `DISTINCT`, `GROUP BY`, `HAVING`, `UNION`, aggregate functions

A view **cannot** be updated if:
- ❌ Based on **multiple tables** (JOIN)
- ❌ Contains calculated/derived columns
- ❌ Uses `TOP` without `ORDER BY`

```sql
-- Updatable view (single table, has PK)
CREATE VIEW vw_ActiveEmployees AS
SELECT EmployeeId, Name, Salary FROM Employees WHERE IsActive = 1;

UPDATE vw_ActiveEmployees SET Salary = 80000 WHERE EmployeeId = 1; -- ✅ Works

-- Non-updatable view (JOIN)
CREATE VIEW vw_EmpDept AS
SELECT e.Name, d.Name AS DeptName FROM Employees e
JOIN Departments d ON e.DeptId = d.DeptId;

UPDATE vw_EmpDept SET DeptName = 'IT'; -- ❌ Error!
```

### INSTEAD OF Trigger on View
> Use `INSTEAD OF UPDATE` trigger to enable updates on non-updatable views.

```sql
CREATE TRIGGER trg_UpdateEmpDept
ON vw_EmpDept
INSTEAD OF UPDATE
AS
BEGIN
    UPDATE Employees SET Name = i.Name FROM INSERTED i
    WHERE Employees.EmployeeId = i.EmployeeId;
END;
```

**Interview Answer:** "A view is updatable if it's based on a single table, contains the primary key, and doesn't use GROUP BY, DISTINCT, or aggregate functions. For complex views with JOINs, you can use an INSTEAD OF trigger to handle updates manually."

---

## 📊 Patterns Quick Reference

| Pattern | One-Liner | Key Benefit |
|---|---|---|
| **DTO** | Simple class to transfer data | Decouples DB model from API response |
| **AutoMapper** | Auto-maps between objects | Removes boilerplate mapping code |
| **Repository** | Abstracts data access behind interface | Testable, swappable data layer |
| **Generic Repository** | One repository for all entities | DRY — no repeated code per entity |
| **Unit of Work** | Groups repos, one SaveChanges | Atomic operations across multiple tables |
| **Service Layer** | Business logic between Controller and Repo | Thin controllers, testable logic |

---

## 🎯 Interview Questions for This Module

| # | Question |
|---|---|
| 1 | What is a DTO? Why do we use it instead of returning entities? |
| 2 | What is AutoMapper? How do you configure a mapping profile? |
| 3 | What is the Repository Pattern? Why is it used? |
| 4 | What is the difference between Repository Pattern and Generic Repository? |
| 5 | What is Unit of Work? How does it work with Repository Pattern? |
| 6 | What is a Service Layer? How is it different from a Repository? |
| 7 | What is the difference between Authentication and Authorization? |
| 8 | What is JWT? How does it work in ASP.NET Core? |
| 9 | What is a Circular Reference in EF Core? How do you fix it? |
| 10 | What is Blocking in SQL Server? How is it different from Deadlock? |
| 11 | How do nested transactions work in SQL Server? |
| 12 | When can you UPDATE through a View in SQL Server? |

---

*[← Back to README](./README.md)*
