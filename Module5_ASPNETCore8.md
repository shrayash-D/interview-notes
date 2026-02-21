# 📌 MODULE 5: ASP.NET CORE 8.0

---

## 5.1 Introduction to ASP.NET Core 8

### What is ASP.NET Core?

> **ASP.NET Core** is a cross-platform, high-performance framework for building modern web applications, APIs, and real-time apps.

### ASP.NET Core vs .NET Framework

| Feature              | ASP.NET Core         | ASP.NET (.NET Framework) |
| -------------------- | -------------------- | ------------------------ |
| Platform             | Cross-platform       | Windows only             |
| Performance          | Very fast            | Slower                   |
| Hosting              | Kestrel, IIS, Docker | IIS only                 |
| Open Source          | ✅ Yes               | ❌ No                    |
| Dependency Injection | Built-in             | Third-party needed       |

---

## 5.2 Project Templates

| Template        | Use Case                             |
| --------------- | ------------------------------------ |
| **MVC**         | Full web app with Views (HTML pages) |
| **Web API**     | RESTful APIs returning JSON          |
| **Minimal API** | Lightweight APIs with minimal code   |
| **Razor Pages** | Page-focused web apps                |
| **Blazor**      | SPA with C# instead of JavaScript    |

### Project Structure

```
MyApp/
├── Controllers/        # Handle HTTP requests
├── Models/             # Data classes
├── Views/              # Razor HTML templates
├── wwwroot/            # Static files (CSS, JS, images)
├── appsettings.json    # Configuration
├── Program.cs          # App entry point
```

---

## 5.3 MVC Architecture

> **MVC = Model + View + Controller**

| Component      | Responsibility                      | Example                 |
| -------------- | ----------------------------------- | ----------------------- |
| **Model**      | Data + business logic               | `Employee.cs`           |
| **View**       | UI (HTML/Razor)                     | `Index.cshtml`          |
| **Controller** | Handles requests, coordinates M & V | `EmployeeController.cs` |

### Request Lifecycle

```
User → URL → Routing → Controller → Action Method → Model (get data) → View (render HTML) → Response to User
```

**Interview Answer:** "In MVC, when a user makes a request, the Router directs it to the right Controller. The Controller calls the Model to get/save data, then passes that data to a View. The View renders the HTML and sends it back to the user."

---

## 5.4 Models

```csharp
// Model with Data Annotations for Validation
public class Employee
{
    public int Id { get; set; }

    [Required(ErrorMessage = "Name is required")]
    [StringLength(100, MinimumLength = 2)]
    public string Name { get; set; }

    [Required]
    [EmailAddress]
    public string Email { get; set; }

    [Range(18, 65)]
    public int Age { get; set; }

    [DataType(DataType.Currency)]
    public decimal Salary { get; set; }
}

// ViewModel — shapes data for a specific view
public class EmployeeListViewModel
{
    public List<Employee> Employees { get; set; }
    public string SearchTerm { get; set; }
    public int TotalCount { get; set; }
}
```

**Interview Answer:** "A ViewModel is a class designed specifically for a view. It can combine data from multiple models and only expose what the view needs. For example, a dashboard ViewModel might combine employee data and department stats."

---

## 5.5 Controllers

```csharp
public class EmployeeController : Controller
{
    private readonly AppDbContext _context;

    // Constructor Injection
    public EmployeeController(AppDbContext context)
    {
        _context = context;
    }

    // GET: /Employee
    public async Task<IActionResult> Index()
    {
        var employees = await _context.Employees.ToListAsync();
        return View(employees);  // Passes data to View
    }

    // GET: /Employee/Details/5
    public async Task<IActionResult> Details(int id)
    {
        var emp = await _context.Employees.FindAsync(id);
        if (emp == null) return NotFound();
        return View(emp);
    }

    // GET: /Employee/Create
    public IActionResult Create() => View();

    // POST: /Employee/Create
    [HttpPost]
    [ValidateAntiForgeryToken]
    public async Task<IActionResult> Create(Employee employee)
    {
        if (ModelState.IsValid)
        {
            await _context.Employees.AddAsync(employee);
            await _context.SaveChangesAsync();
            return RedirectToAction(nameof(Index));
        }
        return View(employee);  // Return with validation errors
    }

    // POST: /Employee/Delete/5
    [HttpPost, ActionName("Delete")]
    [ValidateAntiForgeryToken]
    public async Task<IActionResult> DeleteConfirmed(int id)
    {
        var emp = await _context.Employees.FindAsync(id);
        if (emp != null) _context.Employees.Remove(emp);
        await _context.SaveChangesAsync();
        return RedirectToAction(nameof(Index));
    }
}
```

---

## 5.6 Razor Views

```html
@model List<Employee>
  <h2>Employee List</h2>

  <table class="table">
    <thead>
      <tr>
        <th>Name</th>
        <th>Email</th>
        <th>Salary</th>
      </tr>
    </thead>
    <tbody>
      @foreach (var emp in Model) {
      <tr>
        <td>@emp.Name</td>
        <td>@emp.Email</td>
        <td>@emp.Salary.ToString("C")</td>
      </tr>
      }
    </tbody>
  </table></Employee
>
```

### Layout View (`_Layout.cshtml`)

> Shared template — defines the common structure (navbar, footer) for all pages.

```html
<!DOCTYPE html>
<html>
  <head>
    <title>@ViewBag.Title - My App</title>
  </head>
  <body>
    <nav><!-- common navbar --></nav>

    @RenderBody()
    <!-- Page-specific content goes here -->

    @RenderSection("Scripts", required: false)
  </body>
</html>
```

### Partial View

```csharp
// Call a partial view
@await Html.PartialAsync("_EmployeeCard", employee)
// or
<partial name="_EmployeeCard" model="employee" />
```

---

## 5.7 Routing

### Convention-based Routing

```csharp
// In Program.cs
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
// URL: /Employee/Details/5 → EmployeeController.Details(5)
```

### Attribute Routing

```csharp
[Route("api/[controller]")]
public class EmployeeController : Controller
{
    [HttpGet("{id}")]        // GET api/employee/5
    public IActionResult Get(int id) { ... }

    [HttpPost]               // POST api/employee
    public IActionResult Create(Employee emp) { ... }

    [Route("search/{name}")] // GET api/employee/search/Alice
    public IActionResult Search(string name) { ... }
}
```

### Route Constraints

```csharp
[HttpGet("{id:int}")]           // Only matches integers
[HttpGet("{name:alpha}")]       // Only matches alphabetic
[HttpGet("{id:min(1)}")]        // Minimum value of 1
[HttpGet("{slug:regex(^[a-z0-9-]+$)}")] // Regex pattern
```

### Tag Helpers (Generate URLs)

```html
<a asp-controller="Employee" asp-action="Details" asp-route-id="5"
  >View Details</a
>
<!-- Generates: <a href="/Employee/Details/5">View Details</a> -->

<form asp-action="Create" asp-controller="Employee" method="post"></form>
```

**Interview Answer:** "Convention-based routing uses a pattern like `{controller}/{action}/{id}` and applies to all controllers. Attribute routing lets you define routes directly on the action method, giving you full control over the URL structure."

---

## 5.8 Forms and User Input

```html
@model Employee

<form asp-action="Create" method="post">
  <div class="form-group">
    <label asp-for="Name"></label>
    <input asp-for="Name" class="form-control" />
    <span asp-validation-for="Name" class="text-danger"></span>
  </div>

  <div class="form-group">
    <label asp-for="Email"></label>
    <input asp-for="Email" class="form-control" />
    <span asp-validation-for="Email" class="text-danger"></span>
  </div>

  <button type="submit" class="btn btn-primary">Save</button>
</form>
```

### Model Binding

> ASP.NET Core **automatically maps** form data, query strings, and route data to action method parameters.

```csharp
// Model Binding happens automatically
[HttpPost]
public IActionResult Create(Employee employee)  // Auto-filled from form data
{
    if (ModelState.IsValid)  // Check validation annotations
    {
        // save...
    }
    return View(employee);  // Re-show form with validation errors
}
```

### Model Validation with Data Annotations

```csharp
// Common validation attributes
[Required]                                    // Cannot be null/empty
[StringLength(100, MinimumLength = 2)]        // Length constraint
[EmailAddress]                                // Valid email format
[Phone]                                       // Valid phone format
[Range(1, 100)]                               // Numeric range
[RegularExpression(@"^\d{5}$")]               // Regex validation
[Compare("Password")]                         // Must match another field
[Url]                                         // Valid URL
```

---

## 5.9 State Management

| Method       | Scope                    | Use Case                                             |
| ------------ | ------------------------ | ---------------------------------------------------- |
| **ViewData** | Current request only     | Pass data from Controller to View (dictionary)       |
| **ViewBag**  | Current request only     | Dynamic version of ViewData                          |
| **TempData** | Current + next request   | Pass data between redirects (e.g., success messages) |
| **Session**  | Entire user session      | Shopping cart, logged-in user data                   |
| **Cookies**  | Persistent (client-side) | "Remember me", user preferences                      |

```csharp
// ViewData (Controller → View, current request only)
ViewData["Message"] = "Hello!";
// In View: <p>@ViewData["Message"]</p>

// ViewBag (same as ViewData, but dynamic)
ViewBag.Title = "Employee List";
// In View: <title>@ViewBag.Title</title>

// TempData (survives ONE redirect)
TempData["Success"] = "Employee created successfully!";
return RedirectToAction("Index");
// In Index View: @TempData["Success"]

// Session
HttpContext.Session.SetString("Username", "Alice");
HttpContext.Session.SetInt32("UserId", 42);
var username = HttpContext.Session.GetString("Username");

// Configure Session in Program.cs
builder.Services.AddDistributedMemoryCache();
builder.Services.AddSession(options =>
{
    options.IdleTimeout = TimeSpan.FromMinutes(30);
    options.Cookie.HttpOnly = true;
});
app.UseSession();
```

---

## 5.10 Integrating EF Core with ASP.NET Core

```csharp
// Program.cs — Configure DbContext
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

builder.Services.AddControllersWithViews();
var app = builder.Build();
```

```json
// appsettings.json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=.;Database=MyAppDb;Trusted_Connection=True;TrustServerCertificate=True;"
  }
}
```

---

## 5.11 Dependency Injection (DI)

### What is DI?

> **Dependency Injection** is a design pattern where a class receives its dependencies from outside rather than creating them itself. ASP.NET Core has a **built-in DI container**.

### Service Lifetimes

| Lifetime      | Behavior                                   | Use Case                        |
| ------------- | ------------------------------------------ | ------------------------------- |
| **Transient** | New instance **every time** it's requested | Lightweight, stateless services |
| **Scoped**    | One instance **per HTTP request**          | DbContext, per-request services |
| **Singleton** | One instance for **entire app lifetime**   | Caching, logging, configuration |

```csharp
// Register Services in Program.cs
builder.Services.AddTransient<IEmailService, EmailService>();               // New each time
builder.Services.AddScoped<IEmployeeRepository, EmployeeRepository>();      // Per request
builder.Services.AddSingleton<ICacheService, CacheService>();               // One for all
builder.Services.AddScoped<AppDbContext>();                                  // Per request (default)

// Inject into Controller via constructor
public class EmployeeController : Controller
{
    private readonly IEmployeeRepository _repo;
    private readonly IEmailService _email;

    public EmployeeController(IEmployeeRepository repo, IEmailService email)
    {
        _repo = repo;     // Injected by DI container ✅
        _email = email;   // Injected by DI container ✅
    }
}
```

#### Why DbContext must be Scoped (not Singleton)

```csharp
// ❌ WRONG — Singleton DbContext
builder.Services.AddSingleton<AppDbContext>();
// Problem: One shared DbContext for ALL requests
// → User A's pending changes bleed into User B's request
// → Change tracker state corrupted across concurrent requests
// → Race conditions, data corruption ❌

// ✅ CORRECT — Scoped DbContext (one per HTTP request)
builder.Services.AddScoped<AppDbContext>();
// → Each request gets its own DbContext instance
// → User A's changes are fully isolated from User B's
// → Change tracker is fresh for every request ✅
```

#### Verifying Lifetimes (quick test)

```csharp
// Inject the same service twice in one controller to compare instances
public class TestController : Controller
{
    public TestController(IMyService s1, IMyService s2)
    {
        bool same = ReferenceEquals(s1, s2);
        // Transient  → false  (two different instances even in same request)
        // Scoped     → true   (same instance throughout this request)
        // Singleton  → true   (same instance for entire app lifetime)
    }
}
```

**Interview Answer:** "DI is a pattern where dependencies are injected from outside instead of being created inside a class. ASP.NET Core has a built-in IoC container. There are three lifetimes: Transient (new every time), Scoped (once per request), and Singleton (once for the whole app). This follows the Dependency Inversion Principle."

---

## 5.12 Middleware

### What is Middleware?

> **Middleware** is software that's assembled into an **app pipeline** to handle requests and responses. Each piece of middleware can pass the request to the next one or short-circuit the pipeline.

```csharp
// Program.cs — Middleware pipeline order matters!
var app = builder.Build();

app.UseExceptionHandler("/Error");  // Must be first for error handling
app.UseHsts();
app.UseHttpsRedirection();
app.UseStaticFiles();               // Serve CSS, JS, images
app.UseRouting();
app.UseAuthentication();            // Must be before Authorization
app.UseAuthorization();
app.UseSession();
app.MapControllerRoute(...);

app.Run();

// Custom Middleware
app.Use(async (context, next) =>
{
    // Before next middleware
    Console.WriteLine($"Request: {context.Request.Path}");

    await next();  // Call next middleware

    // After next middleware
    Console.WriteLine($"Response: {context.Response.StatusCode}");
});
```

### app.Use vs app.Run vs app.Map

| Method    | Calls next?                   | Use case                                            |
| --------- | ----------------------------- | --------------------------------------------------- |
| `app.Use` | ✅ Yes (calls `await next()`) | Normal middleware — has logic before AND after next |
| `app.Run` | ❌ No (terminal)              | Short-circuits the pipeline — response ends here    |
| `app.Map` | Branches pipeline             | Route-specific logic for a path prefix              |

```csharp
// app.Use — has a "before" and "after" phase around next()
app.Use(async (context, next) =>
{
    Console.WriteLine("Middleware 1 — BEFORE");
    await next();                                  // Passes to next middleware
    Console.WriteLine("Middleware 1 — AFTER");     // Runs after response comes back
});

app.Use(async (context, next) =>
{
    Console.WriteLine("Middleware 2 — BEFORE");
    await next();
    Console.WriteLine("Middleware 2 — AFTER");
});

// app.Run — TERMINAL, never calls next
app.Run(async context =>
{
    Console.WriteLine("Terminal middleware");
    await context.Response.WriteAsync("Hello from terminal!");
    // Pipeline STOPS here — nothing after this runs
});

// If app.Run is placed before app.Use, the later Use NEVER runs! ⚠️

// Execution order output:
// → Middleware 1 BEFORE
//   → Middleware 2 BEFORE
//     → Terminal middleware (response written)
//   → Middleware 2 AFTER
// → Middleware 1 AFTER


// app.Map — branches for a specific path prefix
app.Map("/api/health", healthApp =>
{
    healthApp.Run(async context =>
    {
        await context.Response.WriteAsync("Healthy ✅");
    });
    // Only runs when path starts with /api/health
});
```

**Interview Answer:** _"`app.Use` adds middleware that can process the request AND response — it calls `await next()` to pass control to the next middleware. `app.Run` is terminal — it handles the request and ends the pipeline without calling next. `app.Map` branches the pipeline for requests matching a specific path prefix. Order matters: `app.Run` before `app.Use` means the `Use` middleware never executes."_

---

## 5.13 Publishing and Deployment

### appsettings for Different Environments

```
appsettings.json                  # Shared settings (base)
appsettings.Development.json      # Dev-specific (overrides base)
appsettings.Production.json       # Prod-specific (overrides base)
```

```csharp
// Program.cs reads env automatically based on ASPNETCORE_ENVIRONMENT
var env = builder.Environment.EnvironmentName; // "Development" or "Production"

if (builder.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();  // Show detailed errors in dev
}
```

### Publishing Options

| Method                  | Command                                                   |
| ----------------------- | --------------------------------------------------------- |
| **Self-Contained**      | `dotnet publish -c Release --self-contained -r win-x64`   |
| **Framework-Dependent** | `dotnet publish -c Release`                               |
| **IIS**                 | Publish → Copy to IIS folder → Configure Application Pool |
| **Azure App Service**   | VS Publish wizard or `az webapp deploy`                   |
| **Docker**              | Create `Dockerfile` → `docker build` → `docker run`       |

### Dockerfile Example

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app
EXPOSE 80

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY . .
RUN dotnet publish -c Release -o /app/publish

FROM base AS final
COPY --from=build /app/publish .
ENTRYPOINT ["dotnet", "MyApp.dll"]
```

---

## 📝 ASP.NET Core Quick Recap

| Topic              | Key Point                                              |
| ------------------ | ------------------------------------------------------ |
| MVC                | Model=data, View=UI, Controller=handles requests       |
| Controller         | Inherits from `Controller`, returns `IActionResult`    |
| Action Result      | `View()`, `Json()`, `RedirectToAction()`, `NotFound()` |
| Razor Syntax       | `@Model`, `@foreach`, `@if`, `asp-for`, `asp-action`   |
| Model Binding      | Automatic mapping of form/route/query data to objects  |
| Model Validation   | `[Required]`, `[Range]`, `ModelState.IsValid`          |
| Convention Routing | `{controller}/{action}/{id?}` pattern                  |
| Attribute Routing  | `[HttpGet("{id}")]` directly on action                 |
| ViewData           | Dict, controller → view, current request               |
| ViewBag            | Dynamic, controller → view, current request            |
| TempData           | Survives one redirect                                  |
| Session            | User session, requires `AddSession()` + `UseSession()` |
| DI Transient       | New instance every injection                           |
| DI Scoped          | One per HTTP request                                   |
| DI Singleton       | One per app lifetime                                   |
| Middleware         | Request/response pipeline, order matters!              |

---

_[← Back to README](./README.md)_
