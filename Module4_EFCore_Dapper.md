# 📌 MODULE 4: ENTITY FRAMEWORK CORE 8.0 & DAPPER

---

## 4.1 What is ORM?

> **ORM (Object-Relational Mapping)** maps **database tables to C# classes** and **rows to objects**. You write C# code instead of SQL.

**Interview Answer:** "ORM is a technique that lets you interact with a database using C# objects instead of writing raw SQL. EF Core is Microsoft's ORM for .NET."

---

## 4.2 EF Core vs EF Framework

| Feature            | EF Core              | EF Framework (EF6)    |
| ------------------ | -------------------- | --------------------- |
| Platform           | Cross-platform       | Windows only          |
| Performance        | Faster               | Slower                |
| .NET Support       | .NET 6/7/8           | .NET Framework only   |
| Approach           | Code-First preferred | Database-First common |
| Active Development | ✅ Yes               | ❌ Maintenance mode   |

---

## 4.3 New Features in EF Core 8

| Feature                        | Description                                                   |
| ------------------------------ | ------------------------------------------------------------- |
| **Complex Types**              | Value objects without identity (like Address)                 |
| **Raw SQL for Unmapped Types** | `SqlQuery<T>` for arbitrary queries                           |
| **Primitive Collections**      | Store `List<int>`, `List<string>` as JSON columns             |
| **Sentinel Values**            | Better handling of default values                             |
| **Lazy Loading Improvements**  | More efficient lazy loading                                   |
| **Bulk Updates & Deletes**     | `ExecuteUpdate()`, `ExecuteDelete()` without loading entities |

---

## 4.4 Setting Up EF Core in .NET 8

```bash
# Install NuGet packages
dotnet add package Microsoft.EntityFrameworkCore
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools
```

```csharp
// DbContext — the "bridge" between your code and database
public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }

    public DbSet<Employee> Employees { get; set; }
    public DbSet<Department> Departments { get; set; }
}

// In Program.cs (.NET 8)
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));
```

### EF Core CLI Commands

```bash
dotnet ef migrations add InitialCreate    # Create migration
dotnet ef database update                  # Apply migration to DB
dotnet ef migrations remove               # Remove last migration
dotnet ef database drop                    # Drop the database
```

---

## 4.5 Defining Entities and Relationships

```csharp
// Entity = C# class mapped to a database table
public class Employee
{
    public int EmployeeId { get; set; }        // Primary Key (by convention)
    public string Name { get; set; }
    public decimal Salary { get; set; }

    public int DepartmentId { get; set; }       // Foreign Key
    public Department Department { get; set; }  // Navigation Property
}

public class Department
{
    public int DepartmentId { get; set; }
    public string Name { get; set; }

    public ICollection<Employee> Employees { get; set; }  // Navigation (One-to-Many)
}
```

---

## 4.6 CRUD Operations with EF Core

```csharp
// CREATE
var emp = new Employee { Name = "Alice", Salary = 75000 };
await _context.Employees.AddAsync(emp);
await _context.SaveChangesAsync();

// READ
var employee = await _context.Employees.FindAsync(1);
var firstEmp = await _context.Employees.FirstOrDefaultAsync(e => e.Name == "Alice");
var allEmps = await _context.Employees.ToListAsync();

// UPDATE
var emp = await _context.Employees.FindAsync(1);
emp.Salary = 80000;         // EF tracks this change automatically
await _context.SaveChangesAsync();

// DELETE
var emp = await _context.Employees.FindAsync(1);
_context.Employees.Remove(emp);
await _context.SaveChangesAsync();

// Bulk Delete (EF Core 8 — no loading needed!)
await _context.Employees.Where(e => e.Salary < 30000).ExecuteDeleteAsync();

// Bulk Update (EF Core 8)
await _context.Employees
    .Where(e => e.Department == "IT")
    .ExecuteUpdateAsync(e => e.SetProperty(x => x.Salary, x => x.Salary * 1.1m));
```

---

## 4.7 LINQ Queries in EF Core

```csharp
// WHERE + OrderBy
var itEmployees = await _context.Employees
    .Where(e => e.Department.Name == "IT")
    .OrderBy(e => e.Name)
    .ToListAsync();

// Projection to DTO
var dtos = await _context.Employees
    .Select(e => new EmployeeDto
    {
        FullName = e.Name,
        DeptName = e.Department.Name
    })
    .ToListAsync();

// Aggregation
var avgSalary = await _context.Employees.AverageAsync(e => e.Salary);
var totalCount = await _context.Employees.CountAsync();
```

---

## 4.8 EF Core Migrations

```bash
# Add a migration (generates code to update DB schema)
dotnet ef migrations add AddSalaryColumn

# Apply to database
dotnet ef database update

# Seed data in migration
protected override void Up(MigrationBuilder migrationBuilder)
{
    migrationBuilder.InsertData(
        table: "Departments",
        columns: new[] { "Name" },
        values: new object[] { "IT" });
}
```

---

## 4.9 Data Loading Strategies

| Strategy             | How It Works                                     | When to Use                       |
| -------------------- | ------------------------------------------------ | --------------------------------- |
| **Eager Loading**    | Loads related data immediately with `.Include()` | You know you'll need related data |
| **Lazy Loading**     | Loads related data on first access (auto)        | Careful — can cause N+1 problem   |
| **Explicit Loading** | Manually load related data with `.Load()`        | You decide when to load           |

```csharp
// Eager Loading
var emps = await _context.Employees
    .Include(e => e.Department)
    .ToListAsync();

// ThenInclude — for nested relationships
var emps = await _context.Employees
    .Include(e => e.Department)
        .ThenInclude(d => d.Manager)
    .ToListAsync();

// Explicit Loading
var emp = await _context.Employees.FindAsync(1);
await _context.Entry(emp).Reference(e => e.Department).LoadAsync();

// Lazy Loading — needs virtual navigation properties + proxy package
```

---

## 4.10 Relationships Configuration

```csharp
// One-to-One
modelBuilder.Entity<Employee>()
    .HasOne(e => e.Address)
    .WithOne(a => a.Employee)
    .HasForeignKey<Address>(a => a.EmployeeId);

// One-to-Many
modelBuilder.Entity<Department>()
    .HasMany(d => d.Employees)
    .WithOne(e => e.Department)
    .HasForeignKey(e => e.DepartmentId);

// Many-to-Many (EF Core 5+ auto-creates join table)
modelBuilder.Entity<Student>()
    .HasMany(s => s.Courses)
    .WithMany(c => c.Students);
```

---

## 4.11 Performance Best Practices

```csharp
// 1. AsNoTracking — for read-only queries (faster, no change tracking overhead)
var emps = await _context.Employees.AsNoTracking().ToListAsync();

// 2. Compiled Queries — pre-compile for repeated execution
private static readonly Func<AppDbContext, int, Task<Employee>> GetById =
    EF.CompileAsyncQuery((AppDbContext ctx, int id) =>
        ctx.Employees.FirstOrDefault(e => e.EmployeeId == id));

// Usage
var emp = await GetById(_context, 1);

// 3. RowVersion for Concurrency (Optimistic Concurrency)
public class Employee
{
    [Timestamp]
    public byte[] RowVersion { get; set; }
    // EF will throw DbUpdateConcurrencyException if another user
    // updated this record since you last read it
}

// 4. Select only what you need (avoid SELECT *)
var names = await _context.Employees
    .Select(e => e.Name)
    .ToListAsync();
```

---

## 4.12 Dapper — Micro ORM

### What is Dapper?

> **Dapper** is a lightweight, high-performance **micro-ORM** for .NET. It extends `IDbConnection` with methods to map SQL results to C# objects.

**Interview Answer:** "Dapper is a micro-ORM that sits on top of ADO.NET. It's much faster than EF Core because it does minimal overhead — just maps SQL results to objects. Created by the Stack Overflow team."

### Dapper vs Entity Framework

| Feature         | Dapper                           | EF Core                       |
| --------------- | -------------------------------- | ----------------------------- |
| Type            | Micro-ORM                        | Full ORM                      |
| Speed           | ⚡ Fastest                       | Slower                        |
| SQL Control     | Full control (raw SQL)           | Generates SQL (LINQ)          |
| Change Tracking | ❌ No                            | ✅ Yes                        |
| Migrations      | ❌ No                            | ✅ Yes                        |
| Learning Curve  | Easy                             | Moderate                      |
| Best For        | Performance-critical, read-heavy | CRUD-heavy, rapid development |

---

### Setting Up Dapper

```bash
dotnet add package Dapper
dotnet add package Microsoft.Data.SqlClient
```

---

### Querying Data with Dapper

```csharp
using Dapper;
using Microsoft.Data.SqlClient;

using var connection = new SqlConnection(connectionString);

// Query Multiple Rows
var employees = await connection.QueryAsync<Employee>(
    "SELECT * FROM Employees WHERE Department = @Dept",
    new { Dept = "IT" });

// Query Single Row
var emp = await connection.QueryFirstOrDefaultAsync<Employee>(
    "SELECT * FROM Employees WHERE Id = @Id", new { Id = 1 });

// QuerySingle — throws if 0 or more than 1 row
var emp = await connection.QuerySingleAsync<Employee>(
    "SELECT * FROM Employees WHERE Id = @Id", new { Id = 1 });

// Query Scalar Value
var count = await connection.ExecuteScalarAsync<int>(
    "SELECT COUNT(*) FROM Employees");

// Query Multiple Result Sets
using var multi = await connection.QueryMultipleAsync(
    "SELECT * FROM Employees; SELECT * FROM Departments;");
var employees = await multi.ReadAsync<Employee>();
var departments = await multi.ReadAsync<Department>();
```

---

### Insert, Update, Delete with Dapper

```csharp
// INSERT
await connection.ExecuteAsync(
    "INSERT INTO Employees (Name, Salary) VALUES (@Name, @Salary)",
    new { Name = "Alice", Salary = 75000 });

// UPDATE
await connection.ExecuteAsync(
    "UPDATE Employees SET Salary = @Salary WHERE Id = @Id",
    new { Id = 1, Salary = 80000 });

// DELETE
await connection.ExecuteAsync(
    "DELETE FROM Employees WHERE Id = @Id", new { Id = 1 });

// Execute Stored Procedure
var result = await connection.QueryAsync<Employee>(
    "usp_GetEmployeesByDept",
    new { DeptName = "IT" },
    commandType: CommandType.StoredProcedure);
```

---

### Parameters & SQL Injection Prevention

```csharp
// ✅ SAFE — Parameterized (Dapper auto-parameterizes)
var emp = await connection.QueryAsync<Employee>(
    "SELECT * FROM Employees WHERE Name = @Name", new { Name = userInput });

// ❌ UNSAFE — SQL Injection vulnerability
var emp = await connection.QueryAsync<Employee>(
    $"SELECT * FROM Employees WHERE Name = '{userInput}'");  // NEVER DO THIS!

// Dynamic Parameters
var parameters = new DynamicParameters();
parameters.Add("@Name", "Alice");
parameters.Add("@MinSalary", 50000);
var result = await connection.QueryAsync<Employee>(sql, parameters);

// WHERE IN
var ids = new[] { 1, 2, 3 };
var emps = await connection.QueryAsync<Employee>(
    "SELECT * FROM Employees WHERE Id IN @Ids", new { Ids = ids });

// Output Parameter
var p = new DynamicParameters();
p.Add("@DeptName", "IT");
p.Add("@Count", dbType: DbType.Int32, direction: ParameterDirection.Output);
await connection.ExecuteAsync("usp_GetEmpCount", p, commandType: CommandType.StoredProcedure);
int count = p.Get<int>("@Count");
```

---

### Relationships with Dapper (SplitOn)

```csharp
// One-to-One / Many-to-One with SplitOn
var sql = @"SELECT e.*, d.* FROM Employees e
            INNER JOIN Departments d ON e.DepartmentId = d.DepartmentId";

var employees = await connection.QueryAsync<Employee, Department, Employee>(
    sql,
    (employee, department) =>
    {
        employee.Department = department;
        return employee;
    },
    splitOn: "DepartmentId"  // Where to split columns between objects
);

// One-to-Many
var sql = @"SELECT d.*, e.* FROM Departments d
            LEFT JOIN Employees e ON d.DepartmentId = e.DepartmentId";

var deptDict = new Dictionary<int, Department>();
var departments = await connection.QueryAsync<Department, Employee, Department>(
    sql,
    (dept, emp) =>
    {
        if (!deptDict.TryGetValue(dept.DepartmentId, out var existingDept))
        {
            existingDept = dept;
            existingDept.Employees = new List<Employee>();
            deptDict.Add(existingDept.DepartmentId, existingDept);
        }
        if (emp != null) existingDept.Employees.Add(emp);
        return existingDept;
    },
    splitOn: "EmployeeId"
);
```

---

### Dapper Plus — Bulk Operations

```csharp
// Bulk Insert
connection.BulkInsert(employeesList);

// Bulk Update
connection.BulkUpdate(employeesList);

// Bulk Delete
connection.BulkDelete(employeesList);

// Bulk Merge (Insert or Update)
connection.BulkMerge(employeesList);
```

---

## 4.13 IEnumerable vs IQueryable vs ICollection

> A very common interview question — these three interfaces look similar but behave very differently, especially with EF Core.

| Interface       | Where filtering happens   | Loads data when?               | Use case                                |
| --------------- | ------------------------- | ------------------------------ | --------------------------------------- |
| **IQueryable**  | In the **database** (SQL) | On `.ToList()` / terminal call | EF Core queries — always prefer this    |
| **IEnumerable** | In **C# memory**          | Immediately when assigned      | In-memory collections, LINQ to Objects  |
| **ICollection** | In **C# memory**          | Already loaded                 | Navigation properties, Add/Remove/Count |

```csharp
// ─── IQueryable<T> — deferred, runs in DB ─────────────────────────────────
IQueryable<Employee> query = _context.Employees
    .Where(e => e.Salary > 50000);              // ← No SQL executed yet!

var result = query.ToList();                    // ← SQL runs NOW
// Generated SQL: SELECT * FROM Employees WHERE Salary > 50000

// Chain more filters — still builds SQL
var filtered = _context.Employees
    .Where(e => e.DepartmentId == 1)
    .OrderBy(e => e.Name)
    .Take(10)
    .ToList();                                  // One efficient SQL query ✅


// ─── IEnumerable<T> — in-memory, loads everything first ──────────────────
IEnumerable<Employee> allEmps = _context.Employees.ToList();  // Loads ALL rows!
var highEarners = allEmps.Where(e => e.Salary > 50000);       // Filters in C# ❌ slow

// ⚠️ Performance trap with a 1 million row table:
// IQueryable → fetches ~10 matching rows from DB ✅
// IEnumerable → fetches 1,000,000 rows into memory, then filters in C# ❌


// ─── ICollection<T> — for navigation properties (already in memory) ───────
public class Department
{
    public int DepartmentId { get; set; }
    public string Name { get; set; }
    public ICollection<Employee> Employees { get; set; }  // Navigation prop ✅
}

// ICollection supports Add, Remove, Count, Contains
department.Employees.Add(newEmployee);
int count = department.Employees.Count;
```

**When to use which:**

| Scenario                               | Use                |
| -------------------------------------- | ------------------ |
| Querying database (filtering, paging)  | `IQueryable<T>` ✅ |
| Already loaded in memory, looping      | `IEnumerable<T>`   |
| Navigation property in EF Core entity  | `ICollection<T>`   |
| Need Add/Remove on a loaded collection | `ICollection<T>`   |

**Interview Answer:** _"IQueryable builds the SQL query and runs it in the database — only when you call ToList() or similar. IEnumerable loads everything into memory first and then filters in C#, which is very inefficient for large tables. ICollection is for in-memory collections that are already loaded — it supports Add/Remove/Count and is typically used for navigation properties in EF Core."_

---

## 4.14 Fluent API Configuration in EF Core

> **Fluent API** configures entity-to-table mappings in code — in the `OnModelCreating` override of your `DbContext`. It is more powerful than Data Annotations and keeps entity classes clean.

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Employee>(entity =>
    {
        // Table and schema name
        entity.ToTable("tbl_Employees", schema: "hr");

        // Column configuration
        entity.Property(e => e.Name)
              .IsRequired()
              .HasMaxLength(100)
              .HasColumnName("EmployeeName");

        entity.Property(e => e.Salary)
              .HasColumnType("decimal(18,2)")
              .HasDefaultValue(0);

        entity.Property(e => e.CreatedAt)
              .HasDefaultValueSql("GETUTCDATE()");  // DB-side default

        // Unique index
        entity.HasIndex(e => e.Email)
              .IsUnique();

        // Composite index
        entity.HasIndex(e => new { e.DepartmentId, e.Name });

        // Relationship: Many employees → One department
        entity.HasOne(e => e.Department)
              .WithMany(d => d.Employees)
              .HasForeignKey(e => e.DepartmentId)
              .OnDelete(DeleteBehavior.Restrict);   // Prevent cascade delete
    });

    // Composite Primary Key (can only be done with Fluent API, not Data Annotations)
    modelBuilder.Entity<OrderItem>()
        .HasKey(oi => new { oi.OrderId, oi.ProductId });
}
```

**Fluent API vs Data Annotations:**

| Aspect              | Data Annotations                   | Fluent API                               |
| ------------------- | ---------------------------------- | ---------------------------------------- |
| Location            | On the entity class (attributes)   | In `OnModelCreating` in DbContext        |
| Entity cleanliness  | ❌ Entity has persistence concerns | ✅ Entity is a clean POCO                |
| Power / flexibility | Limited                            | Full control                             |
| Composite keys      | ❌ Not supported                   | ✅ Supported                             |
| Preferred when      | Simple projects, quick setup       | Production apps, DDD, clean architecture |

**Use Fluent API when:**

- You cannot or don't want to modify the entity class
- You need composite keys
- You need complex relationship configuration
- You want to keep entities as clean POCOs

---

## 📝 EF Core & Dapper Quick Recap

| Topic                | Key Point                                    |
| -------------------- | -------------------------------------------- |
| ORM                  | Maps DB tables ↔ C# classes                  |
| DbContext            | Bridge between code and DB                   |
| `AddAsync`           | Insert new record                            |
| `FindAsync`          | Find by primary key                          |
| `SaveChangesAsync`   | Commit all pending changes                   |
| `ExecuteDeleteAsync` | Bulk delete (EF Core 8)                      |
| `ExecuteUpdateAsync` | Bulk update (EF Core 8)                      |
| Eager Loading        | `.Include()` — load related data immediately |
| Lazy Loading         | Load on first access — watch for N+1!        |
| `AsNoTracking`       | Read-only query, faster performance          |
| Dapper `Query<T>`    | Map rows to C# objects                       |
| Dapper `Execute`     | INSERT / UPDATE / DELETE                     |
| Dapper `SplitOn`     | Map JOINs to related objects                 |
| SQL Injection        | Use `@params`, never string concatenation    |

---

_[← Back to README](./README.md)_
