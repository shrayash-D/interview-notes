# 📌 MODULE 2: C# BEST PRACTICES — SOLID PRINCIPLES

---

## What are SOLID Principles?

> **SOLID** is a set of **5 design principles** that help you write **clean, maintainable, and scalable** object-oriented code.

**Why SOLID?**

- Makes code **easy to understand** and **modify**
- Reduces **bugs** when adding new features
- Makes code **testable**
- Promotes **loose coupling**

---

## S — Single Responsibility Principle (SRP)

> **"A class should have only ONE reason to change."**

Each class should do **one thing only**.

```csharp
// ❌ BAD — Class does too many things
public class Employee
{
    public void CalculateSalary() { /* salary logic */ }
    public void SaveToDatabase() { /* DB logic */ }
    public void SendEmail() { /* email logic */ }
}

// ✅ GOOD — Each class has one job
public class SalaryCalculator
{
    public decimal Calculate(Employee emp) { /* salary logic */ }
}

public class EmployeeRepository
{
    public void Save(Employee emp) { /* DB logic */ }
}

public class EmailService
{
    public void SendEmail(string to, string body) { /* email logic */ }
}
```

**Interview Answer:** "SRP means a class should have only one responsibility or one reason to change. For example, separate your business logic, data access, and email sending into different classes."

---

## O — Open/Closed Principle (OCP)

> **"Classes should be OPEN for extension, but CLOSED for modification."**

You should be able to add new behavior **without changing** existing code.

```csharp
// ❌ BAD — Must modify class for each new shape
public class AreaCalculator
{
    public double Calculate(string shape, double r)
    {
        if (shape == "Circle") return 3.14 * r * r;
        if (shape == "Square") return r * r;
        // Adding new shape = modifying this class ❌
    }
}

// ✅ GOOD — Extend by adding new classes
public interface IShape
{
    double Area();
}

public class Circle : IShape
{
    public double Radius { get; set; }
    public double Area() => 3.14 * Radius * Radius;
}

public class Square : IShape
{
    public double Side { get; set; }
    public double Area() => Side * Side;
}

// Adding Triangle? Just create a new class — no existing code changes!
public class Triangle : IShape
{
    public double Base { get; set; }
    public double Height { get; set; }
    public double Area() => 0.5 * Base * Height;
}
```

**Interview Answer:** "OCP means you should extend behavior by adding new code (new classes), not by modifying existing code. Use interfaces and inheritance to achieve this."

---

## L — Liskov Substitution Principle (LSP)

> **"A child class should be able to replace its parent class without breaking anything."**

If class B inherits from class A, you should be able to use B wherever A is expected.

```csharp
// ❌ BAD — Penguin breaks the contract
public class Bird
{
    public virtual void Fly() { Console.WriteLine("Flying!"); }
}

public class Penguin : Bird
{
    public override void Fly()
    {
        throw new Exception("Penguins can't fly!"); // ❌ Breaks LSP
    }
}

// ✅ GOOD — Separate the concerns
public interface IBird { void Eat(); }
public interface IFlyingBird : IBird { void Fly(); }

public class Sparrow : IFlyingBird
{
    public void Eat() { /* eat */ }
    public void Fly() { /* fly */ }
}

public class Penguin : IBird
{
    public void Eat() { /* eat */ }
    // No Fly() — Penguins don't fly, no contract broken ✅
}
```

**Interview Answer:** "LSP means if you substitute a parent class with a child class, the program should still work correctly. If a subclass can't fulfill the parent's contract, you should redesign the hierarchy."

---

## I — Interface Segregation Principle (ISP)

> **"Don't force a class to implement methods it doesn't use."**

Split big interfaces into smaller, focused ones.

```csharp
// ❌ BAD — Too many methods in one interface
public interface IWorker
{
    void Work();
    void Eat();
    void Sleep();
}

public class Robot : IWorker
{
    public void Work() { /* works */ }
    public void Eat() { /* Robots don't eat! */ } // ❌ Forced to implement
    public void Sleep() { /* Robots don't sleep! */ } // ❌ Forced to implement
}

// ✅ GOOD — Split interfaces
public interface IWorkable { void Work(); }
public interface IEatable { void Eat(); }
public interface ISleepable { void Sleep(); }

public class Human : IWorkable, IEatable, ISleepable
{
    public void Work() { }
    public void Eat() { }
    public void Sleep() { }
}

public class Robot : IWorkable
{
    public void Work() { } // Only implements what it needs ✅
}
```

**Interview Answer:** "ISP means interfaces should be small and specific. Instead of one big interface, create multiple smaller ones so classes only implement what they actually need."

---

## D — Dependency Inversion Principle (DIP)

> **"High-level modules should NOT depend on low-level modules. Both should depend on ABSTRACTIONS (interfaces)."**

```csharp
// ❌ BAD — Tightly coupled to SqlDatabase
public class OrderService
{
    private SqlDatabase _db = new SqlDatabase(); // ❌ Direct dependency

    public void SaveOrder(Order order)
    {
        _db.Save(order);
    }
}

// ✅ GOOD — Depends on abstraction
public interface IDatabase
{
    void Save(Order order);
}

public class SqlDatabase : IDatabase
{
    public void Save(Order order) { /* SQL save */ }
}

public class MongoDatabase : IDatabase
{
    public void Save(Order order) { /* Mongo save */ }
}

public class OrderService
{
    private readonly IDatabase _db;

    public OrderService(IDatabase db) // ✅ Injected via constructor
    {
        _db = db;
    }

    public void SaveOrder(Order order)
    {
        _db.Save(order);
    }
}
```

**Interview Answer:** "DIP means high-level classes should depend on interfaces, not on concrete implementations. This allows you to easily swap implementations (e.g., switch from SQL Server to MongoDB) without changing business logic. It's the foundation of Dependency Injection."

---

## 📊 SOLID Quick Reference Table

| Principle                 | Letter | One-liner                                    | Key Idea                              |
| ------------------------- | ------ | -------------------------------------------- | ------------------------------------- |
| **S**ingle Responsibility | S      | One class = one job                          | Don't mix concerns                    |
| **O**pen/Closed           | O      | Add features without changing existing code  | Use interfaces/inheritance            |
| **L**iskov Substitution   | L      | Child replaces parent without breaking       | Subclass must honor parent's contract |
| **I**nterface Segregation | I      | Small, focused interfaces                    | Don't force unused methods            |
| **D**ependency Inversion  | D      | Depend on abstractions, not concrete classes | Use interfaces + DI                   |

---

## 🎯 Interview Tips for SOLID

- **S** → Think: "Does this class have more than one job?" If yes, split it.
- **O** → Think: "Am I modifying existing code to add a feature?" If yes, use polymorphism.
- **L** → Think: "Does my subclass break any behavior promised by the parent?" If yes, redesign.
- **I** → Think: "Are there methods in this interface that some implementors don't need?" If yes, split.
- **D** → Think: "Is my class creating its own dependencies with `new`?" If yes, inject them.

---

_[← Back to README](./README.md)_
