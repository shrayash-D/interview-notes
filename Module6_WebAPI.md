# 📗 Module 6: ASP.NET Core 8 Web API

> **Focus:** Building production-grade REST APIs — from basic CRUD to JWT auth, Swagger docs, Serilog logging, and SOAP services.

---

## 6.1 What is a Web API?

> A **Web API** is an application that exposes functionality over HTTP so other apps (mobile, web, other services) can consume it.

| Term | Meaning |
|---|---|
| **API** | Application Programming Interface — a contract for how systems talk |
| **Web API** | API that communicates over HTTP/HTTPS |
| **REST API** | Web API following REST architectural principles |
| **Endpoint** | A URL that the API exposes (e.g., `GET /api/employees`) |

---

## 6.2 REST vs SOAP

| Feature | REST | SOAP |
|---|---|---|
| **Full form** | Representational State Transfer | Simple Object Access Protocol |
| **Format** | JSON (mostly), XML | XML only |
| **Protocol** | HTTP | HTTP, SMTP, TCP |
| **Speed** | Faster (lightweight) | Slower (heavy XML) |
| **Standards** | Flexible | Strict (WSDL, WS-Security) |
| **Stateless?** | ✅ Yes | ❌ Can be stateful |
| **Use case** | Public APIs, mobile apps | Enterprise, banking, legacy systems |
| **Error handling** | HTTP status codes | SOAP Fault element |

**Interview Answer:** "REST is lightweight, uses JSON, and works over HTTP with standard verbs (GET, POST, PUT, DELETE). SOAP is stricter, uses XML only, and has built-in security standards. REST is preferred for modern APIs; SOAP is still used in enterprise/banking systems."

---

## 6.3 REST Principles (Constraints)

| Principle | What it means |
|---|---|
| **Stateless** | Each request is self-contained — server stores no client state |
| **Client-Server** | Client and server are separate — client handles UI, server handles data |
| **Cacheable** | Responses can be cached to improve performance |
| **Uniform Interface** | Consistent URLs, HTTP verbs, and response formats |
| **Layered System** | Client doesn't know if it's talking directly to the server |
| **Code on Demand** | Server can send executable code (optional) |

---

## 6.4 Setting Up ASP.NET Core 8 Web API

```bash
# Create new Web API project
dotnet new webapi -n MyApi
cd MyApi
dotnet run
```

### Project Structure
```
MyApi/
├── Controllers/          ← API controllers go here
├── Models/               ← Entity classes
├── DTOs/                 ← Data Transfer Objects
├── Services/             ← Business logic
├── Repositories/         ← Data access layer
├── Middleware/           ← Custom middleware
├── Program.cs            ← App entry point + DI setup
├── appsettings.json      ← Config (connection strings, JWT settings)
└── MyApi.csproj
```

### Program.cs (Minimal hosting model)
```csharp
var builder = WebApplication.CreateBuilder(args);

// Add services
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();
builder.Services.AddDbContext<AppDbContext>(opt =>
    opt.UseSqlServer(builder.Configuration.GetConnectionString("Default")));

var app = builder.Build();

// Middleware pipeline
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

---

## 6.5 Creating Controllers and Actions

### Basic API Controller
```csharp
[ApiController]                    // Enables model binding, automatic 400 responses
[Route("api/[controller]")]        // Route = api/employees
public class EmployeesController : ControllerBase  // ControllerBase = no View support
{
    private readonly AppDbContext _context;

    public EmployeesController(AppDbContext context)
    {
        _context = context;
    }

    // GET api/employees
    [HttpGet]
    public async Task<ActionResult<IEnumerable<EmployeeDto>>> GetAll()
    {
        var employees = await _context.Employees.ToListAsync();
        return Ok(employees);
    }

    // GET api/employees/5
    [HttpGet("{id}")]
    public async Task<ActionResult<EmployeeDto>> GetById(int id)
    {
        var emp = await _context.Employees.FindAsync(id);
        if (emp == null) return NotFound();  // 404
        return Ok(emp);
    }

    // POST api/employees
    [HttpPost]
    public async Task<ActionResult<EmployeeDto>> Create(CreateEmployeeDto dto)
    {
        var emp = new Employee { Name = dto.Name, Email = dto.Email };
        _context.Employees.Add(emp);
        await _context.SaveChangesAsync();
        return CreatedAtAction(nameof(GetById), new { id = emp.Id }, emp);  // 201
    }

    // PUT api/employees/5
    [HttpPut("{id}")]
    public async Task<IActionResult> Update(int id, UpdateEmployeeDto dto)
    {
        var emp = await _context.Employees.FindAsync(id);
        if (emp == null) return NotFound();

        emp.Name = dto.Name;
        emp.Email = dto.Email;
        await _context.SaveChangesAsync();
        return NoContent();  // 204
    }

    // DELETE api/employees/5
    [HttpDelete("{id}")]
    public async Task<IActionResult> Delete(int id)
    {
        var emp = await _context.Employees.FindAsync(id);
        if (emp == null) return NotFound();

        _context.Employees.Remove(emp);
        await _context.SaveChangesAsync();
        return NoContent();  // 204
    }
}
```

### HTTP Status Codes Cheat Sheet
| Status | Method | Meaning |
|---|---|---|
| `200 OK` | GET | Success, returns data |
| `201 Created` | POST | Resource created |
| `204 No Content` | PUT/DELETE | Success, no body |
| `400 Bad Request` | Any | Invalid input |
| `401 Unauthorized` | Any | Not authenticated |
| `403 Forbidden` | Any | Authenticated but not allowed |
| `404 Not Found` | GET/PUT/DELETE | Resource doesn't exist |
| `500 Internal Server Error` | Any | Server crash |

---

## 6.6 Attribute Routing and Query Parameters

```csharp
// Attribute route with constraint
[HttpGet("{id:int}")]       // Only matches integers
[HttpGet("{name:alpha}")]   // Only matches letters

// Multiple routes for same action
[HttpGet("all")]
[HttpGet("list")]
public IActionResult GetAll() { ... }

// Query parameters  — GET /api/employees?page=1&size=10&search=john
[HttpGet]
public async Task<IActionResult> GetAll(
    [FromQuery] int page = 1,
    [FromQuery] int size = 10,
    [FromQuery] string? search = null)
{
    var query = _context.Employees.AsQueryable();

    if (!string.IsNullOrEmpty(search))
        query = query.Where(e => e.Name.Contains(search));

    var result = await query
        .Skip((page - 1) * size)
        .Take(size)
        .ToListAsync();

    return Ok(result);
}

// Route + Query together
[HttpGet("department/{deptId}/employees")]
public async Task<IActionResult> GetByDept(int deptId, [FromQuery] bool activeOnly = true) { ... }

// From different sources
public IActionResult Create(
    [FromRoute] int id,           // From URL path
    [FromQuery] string format,    // From query string
    [FromBody] CreateDto dto,     // From request body (JSON)
    [FromHeader] string apiKey)   // From request header
```

---

## 6.7 JSON Formatting and Serialization

```csharp
// Program.cs — Configure JSON options
builder.Services.AddControllers()
    .AddJsonOptions(options =>
    {
        // camelCase property names in JSON response
        options.JsonSerializerOptions.PropertyNamingPolicy = JsonNamingPolicy.CamelCase;

        // Ignore null values in response
        options.JsonSerializerOptions.DefaultIgnoreCondition =
            JsonIgnoreCondition.WhenWritingNull;

        // Handle circular references
        options.JsonSerializerOptions.ReferenceHandler = ReferenceHandler.IgnoreCycles;

        // Pretty print (dev only)
        options.JsonSerializerOptions.WriteIndented = true;
    });

// Property attributes
public class EmployeeDto
{
    [JsonPropertyName("full_name")]   // Custom JSON key name
    public string Name { get; set; }

    [JsonIgnore]                       // Never include in response
    public string PasswordHash { get; set; }

    [JsonConverter(typeof(DateOnlyJsonConverter))]
    public DateOnly BirthDate { get; set; }
}
```

---

## 6.8 Global Exception Handling

> Instead of wrapping every action in try/catch, use a **global exception handler** so errors are caught in one place.

```csharp
// Option 1: Custom Middleware (recommended)
public class GlobalExceptionMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<GlobalExceptionMiddleware> _logger;

    public GlobalExceptionMiddleware(RequestDelegate next,
        ILogger<GlobalExceptionMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            await _next(context);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Unhandled exception: {Message}", ex.Message);
            await HandleExceptionAsync(context, ex);
        }
    }

    private static async Task HandleExceptionAsync(HttpContext context, Exception ex)
    {
        context.Response.ContentType = "application/json";

        var (statusCode, message) = ex switch
        {
            NotFoundException => (404, ex.Message),
            ValidationException => (400, ex.Message),
            UnauthorizedAccessException => (401, "Unauthorized"),
            _ => (500, "An unexpected error occurred")
        };

        context.Response.StatusCode = statusCode;

        var response = new { status = statusCode, message };
        await context.Response.WriteAsJsonAsync(response);
    }
}

// Register in Program.cs — MUST be first!
app.UseMiddleware<GlobalExceptionMiddleware>();

// Custom exception classes
public class NotFoundException : Exception
{
    public NotFoundException(string message) : base(message) { }
}

// Use in service
public async Task<Employee> GetByIdAsync(int id)
{
    var emp = await _context.Employees.FindAsync(id);
    if (emp == null)
        throw new NotFoundException($"Employee with id {id} not found.");
    return emp;
}

// Option 2: Built-in UseExceptionHandler (simpler)
app.UseExceptionHandler(appError =>
{
    appError.Run(async context =>
    {
        context.Response.StatusCode = 500;
        context.Response.ContentType = "application/json";
        var error = context.Features.Get<IExceptionHandlerFeature>();
        if (error != null)
            await context.Response.WriteAsJsonAsync(new { message = error.Error.Message });
    });
});
```

**Interview Answer:** "Global exception handling catches all unhandled exceptions in one middleware instead of scattering try/catch everywhere. I create a custom middleware, log the error with Serilog, and return a consistent JSON error response with the right HTTP status code."

---

## 6.9 Logging with Serilog

> **Serilog** is a structured logging library for .NET. It logs data as structured objects (not just plain text), making it easy to search and filter in tools like Seq, ELK, or Azure App Insights.

```bash
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Sinks.Console
dotnet add package Serilog.Sinks.File
```

```csharp
// Program.cs — Replace default logging with Serilog
using Serilog;

Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Information()
    .MinimumLevel.Override("Microsoft", LogEventLevel.Warning)  // Quiet down ASP.NET logs
    .WriteTo.Console()
    .WriteTo.File("logs/app-.log",
        rollingInterval: RollingInterval.Day,    // New file each day
        retainedFileCountLimit: 7)               // Keep 7 days of logs
    .CreateLogger();

builder.Host.UseSerilog();  // Replace built-in logger with Serilog

// Use in classes
public class EmployeeService
{
    private readonly ILogger<EmployeeService> _logger;

    public EmployeeService(ILogger<EmployeeService> logger)
    {
        _logger = logger;
    }

    public async Task<Employee> GetByIdAsync(int id)
    {
        _logger.LogInformation("Fetching employee with id {EmployeeId}", id);

        var emp = await _context.Employees.FindAsync(id);
        if (emp == null)
        {
            _logger.LogWarning("Employee {EmployeeId} not found", id);
            throw new NotFoundException($"Employee {id} not found");
        }

        return emp;
    }
}

// Log levels (least → most severe)
_logger.LogTrace("Very detailed diagnostic info");
_logger.LogDebug("Debug info for development");
_logger.LogInformation("Normal operations: {EmployeeId}", id);
_logger.LogWarning("Something unexpected but not breaking: {Count}", count);
_logger.LogError(ex, "Error occurred: {Message}", ex.Message);
_logger.LogCritical("App is going down: {Message}", ex.Message);
```

**Interview Answer:** "Serilog is a structured logging library — instead of logging plain text strings, it logs data as key-value pairs. This makes it easy to search logs (e.g., filter by `EmployeeId`). I typically configure it to write to Console in dev and to rolling files or Azure App Insights in production."

---

## 6.10 JWT Authentication and Authorization

```bash
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
```

```csharp
// appsettings.json
{
  "Jwt": {
    "Key": "YourSuperSecretKeyThatIsAtLeast32CharactersLong!",
    "Issuer": "MyApi",
    "Audience": "MyApiUsers",
    "ExpiresInMinutes": 60
  }
}

// Program.cs — Register JWT
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer           = true,
            ValidateAudience         = true,
            ValidateLifetime         = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer              = builder.Configuration["Jwt:Issuer"],
            ValidAudience            = builder.Configuration["Jwt:Audience"],
            IssuerSigningKey         = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"]))
        };
    });

builder.Services.AddAuthorization();

app.UseAuthentication();   // Who are you?
app.UseAuthorization();    // What can you do?

// Auth Controller — Login endpoint
[ApiController]
[Route("api/[controller]")]
public class AuthController : ControllerBase
{
    private readonly IConfiguration _config;

    public AuthController(IConfiguration config) { _config = config; }

    [HttpPost("login")]
    public IActionResult Login([FromBody] LoginDto dto)
    {
        // In real app: validate against DB
        if (dto.Username != "admin" || dto.Password != "pass123")
            return Unauthorized(new { message = "Invalid credentials" });

        var token = GenerateToken(dto.Username, "Admin");
        return Ok(new { token });
    }

    private string GenerateToken(string username, string role)
    {
        var claims = new[]
        {
            new Claim(ClaimTypes.Name, username),
            new Claim(ClaimTypes.Role, role),
            new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString())
        };

        var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_config["Jwt:Key"]));
        var creds = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);

        var token = new JwtSecurityToken(
            issuer: _config["Jwt:Issuer"],
            audience: _config["Jwt:Audience"],
            claims: claims,
            expires: DateTime.UtcNow.AddMinutes(double.Parse(_config["Jwt:ExpiresInMinutes"])),
            signingCredentials: creds
        );

        return new JwtSecurityTokenHandler().WriteToken(token);
    }
}

// Protect endpoints
[Authorize]                          // Must be logged in
[HttpGet]
public IActionResult GetAll() { ... }

[Authorize(Roles = "Admin")]         // Must be Admin
[HttpDelete("{id}")]
public IActionResult Delete(int id) { ... }

[AllowAnonymous]                     // Anyone can call this
[HttpGet("public")]
public IActionResult PublicEndpoint() { ... }
```

### How JWT Works (Flow)
```
1. Client → POST /api/auth/login  { username, password }
2. Server validates → Returns JWT token (string)
3. Client stores token (localStorage / memory)
4. Client → GET /api/employees  Authorization: Bearer <token>
5. Server validates token → Returns data if valid
```

**JWT Structure:** `header.payload.signature`
```
eyJhbGciOiJIUzI1NiJ9  ← header (base64: algorithm)
.eyJzdWIiOiJhZG1pbiJ9 ← payload (base64: claims like name, role, expiry)
.SflKxwRJSMeKKF2QT4fw  ← signature (proves it wasn't tampered)
```

---

## 6.11 Securing APIs with API Keys and OAuth

### API Key Authentication
```csharp
// Custom middleware to validate API key
public class ApiKeyMiddleware
{
    private readonly RequestDelegate _next;
    private const string API_KEY_HEADER = "X-API-Key";

    public ApiKeyMiddleware(RequestDelegate next) { _next = next; }

    public async Task InvokeAsync(HttpContext context)
    {
        if (!context.Request.Headers.TryGetValue(API_KEY_HEADER, out var extractedApiKey))
        {
            context.Response.StatusCode = 401;
            await context.Response.WriteAsync("API Key missing");
            return;
        }

        var appSettings = context.RequestServices.GetRequiredService<IConfiguration>();
        var apiKey = appSettings.GetValue<string>("ApiKey");

        if (!apiKey.Equals(extractedApiKey))
        {
            context.Response.StatusCode = 403;
            await context.Response.WriteAsync("Invalid API Key");
            return;
        }

        await _next(context);
    }
}
```

### OAuth 2.0 Overview
```
OAuth 2.0 is an authorization framework — it lets users grant third-party apps
access to their data WITHOUT sharing their password.

Flow:
1. User clicks "Login with Google"
2. App redirects to Google's auth server
3. User logs in at Google, grants permission
4. Google redirects back with Authorization Code
5. App exchanges code for Access Token
6. App uses Access Token to call Google APIs

Grant Types:
- Authorization Code ← Most secure (web apps)
- Client Credentials ← Machine-to-machine (no user)
- Implicit           ← Deprecated
- Password           ← Deprecated
```

---

## 6.12 Swagger / OpenAPI Documentation

> **Swagger** auto-generates interactive API documentation from your code. Developers can test endpoints directly in the browser.

```bash
dotnet add package Swashbuckle.AspNetCore
```

```csharp
// Program.cs
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen(options =>
{
    options.SwaggerDoc("v1", new OpenApiInfo
    {
        Title = "My API",
        Version = "v1",
        Description = "Employee Management API"
    });

    // Add JWT support in Swagger UI
    options.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme
    {
        Description = "JWT Authorization header. Enter: Bearer {token}",
        Name = "Authorization",
        In = ParameterLocation.Header,
        Type = SecuritySchemeType.ApiKey,
        Scheme = "Bearer"
    });

    options.AddSecurityRequirement(new OpenApiSecurityRequirement
    {
        {
            new OpenApiSecurityScheme
            {
                Reference = new OpenApiReference { Type = ReferenceType.SecurityScheme, Id = "Bearer" }
            },
            Array.Empty<string>()
        }
    });

    // Include XML comments from code
    var xmlFile = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";
    options.IncludeXmlComments(Path.Combine(AppContext.BaseDirectory, xmlFile));
});

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI(c => c.SwaggerEndpoint("/swagger/v1/swagger.json", "My API v1"));
}

// XML comment on action = shows in Swagger
/// <summary>Get an employee by ID</summary>
/// <param name="id">The employee's unique identifier</param>
/// <returns>Employee data</returns>
/// <response code="200">Employee found</response>
/// <response code="404">Employee not found</response>
[ProducesResponseType(typeof(EmployeeDto), 200)]
[ProducesResponseType(404)]
[HttpGet("{id}")]
public async Task<ActionResult<EmployeeDto>> GetById(int id) { ... }
```

---

## 6.13 Consuming and Creating SOAP Services

### What is WCF?
> **WCF (Windows Communication Foundation)** is a Microsoft framework for building SOAP/XML web services. In .NET Core/.NET 8, you use `CoreWCF` for creating SOAP services and `System.ServiceModel` for consuming them.

### Consuming an Existing SOAP Service
```bash
# Add reference to WSDL (generates proxy classes)
dotnet add package System.ServiceModel.Http
# Or use Visual Studio: Add → Service Reference → WCF Web Service
```

```csharp
// Auto-generated proxy from WSDL
var client = new WeatherSoapClient(
    WeatherSoapClient.EndpointConfiguration.WeatherSoap,
    new EndpointAddress("http://example.com/weather.asmx")
);

var result = await client.GetWeatherAsync("London");
Console.WriteLine(result.Body.GetWeatherResult);

// Always close the client
await client.CloseAsync();
```

### Creating a SOAP Service with CoreWCF
```bash
dotnet add package CoreWCF.Http
```

```csharp
// 1. Define Service Contract (interface with attributes)
[ServiceContract]
public interface IEmployeeService
{
    [OperationContract]
    Task<Employee> GetEmployeeAsync(int id);

    [OperationContract]
    Task<bool> CreateEmployeeAsync(Employee employee);
}

// 2. Implement the service
public class EmployeeService : IEmployeeService
{
    public async Task<Employee> GetEmployeeAsync(int id)
    {
        return await _context.Employees.FindAsync(id);
    }

    public async Task<bool> CreateEmployeeAsync(Employee employee)
    {
        _context.Employees.Add(employee);
        await _context.SaveChangesAsync();
        return true;
    }
}

// 3. Register in Program.cs
builder.Services.AddServiceModelServices();
builder.Services.AddScoped<IEmployeeService, EmployeeService>();

app.UseServiceModel(serviceBuilder =>
{
    serviceBuilder.AddService<EmployeeService>();
    serviceBuilder.AddServiceEndpoint<EmployeeService, IEmployeeService>(
        new BasicHttpBinding(),
        "/EmployeeService.svc");
});
```

---

## 6.14 Postman for API Testing

### Common Postman Workflows

```
1. Create a Collection for your API
2. Add requests (GET, POST, PUT, DELETE)
3. Set Environment Variables ({{baseUrl}}, {{token}})
4. Use Pre-request Scripts to auto-set tokens

# GET  {{baseUrl}}/api/employees
# POST {{baseUrl}}/api/employees
#   Body (raw JSON):
#   { "name": "John", "email": "john@example.com", "salary": 50000 }

# Authorization tab → Bearer Token → {{token}}
```

### Postman Test Scripts (JavaScript)
```javascript
// Tests tab — run after each request
pm.test("Status is 200", () => pm.response.to.have.status(200));
pm.test("Response is JSON", () => pm.response.to.be.json);
pm.test("Has employee name", () => {
    const json = pm.response.json();
    pm.expect(json.name).to.not.be.empty;
});

// Save token from login response to environment
pm.test("Save auth token", () => {
    const json = pm.response.json();
    pm.environment.set("token", json.token);
});
```

---

## 🎯 Interview Questions for Module 6

| # | Question |
|---|---|
| 1 | What is a REST API? What are the REST constraints? |
| 2 | REST vs SOAP — when would you use each? |
| 3 | What is the difference between `ControllerBase` and `Controller`? |
| 4 | What HTTP status codes do you return for POST, PUT, DELETE? |
| 5 | What is the difference between `[FromBody]`, `[FromQuery]`, `[FromRoute]`? |
| 6 | How do you handle exceptions globally in ASP.NET Core Web API? |
| 7 | What is Serilog? How is it different from built-in logging? |
| 8 | How does JWT authentication work in ASP.NET Core? |
| 9 | What is the difference between Authentication and Authorization? |
| 10 | What is Swagger/OpenAPI? How do you enable it? |
| 11 | How do you add JWT support to the Swagger UI? |
| 12 | What is an API Key? How do you validate it? |
| 13 | What is OAuth 2.0? What are the grant types? |
| 14 | What is WCF? How do you consume a SOAP service in .NET 8? |

---

*[← Back to README](./README.md) | [Next Module →](./Module7_Microservices.md)*
