---
name: solution-architecture
description: Use when designing application structure, implementing layered architecture, applying DDD patterns, or making architectural decisions about separation of concerns
---

# Solution Architecture

## Overview

Solution architecture defines how code is organized into layers and components. This skill covers **Domain-Driven Design (DDD)** layered architecture and essential coding principles like **DRY** and **SOLID**.

**Core principle:** The domain is the heart of your application. All other layers exist to support it.

## When to Use

- Starting a new project or module
- Refactoring monolithic code into layers
- Deciding where business logic belongs
- Implementing repositories, services, or domain models
- Reviewing code for architectural violations

**Not for:** Simple CRUD apps, scripts, or prototypes where layering adds unnecessary complexity.

## The Four Layers

```
┌─────────────────────────────────────┐
│         PRESENTATION LAYER          │  ← UI, Controllers, ViewModels
├─────────────────────────────────────┤
│         APPLICATION LAYER           │  ← Use Cases, Orchestration, CQRS
├─────────────────────────────────────┤
│           DOMAIN LAYER              │  ← Entities, Value Objects, Domain Services
├─────────────────────────────────────┤
│        INFRASTRUCTURE LAYER         │  ← Database, External APIs, Messaging
└─────────────────────────────────────┘
```

**Dependency Rule:** Each layer only depends on the layer directly inside it. Domain is innermost and has NO external dependencies.

### Presentation Layer

**Responsibility:** User interaction and display

**Contains:**
- UI components (pages, views, components)
- Controllers / API endpoints
- ViewModels / DTOs for display
- Input validation (format only, not business rules)

**Rules:**
- No business logic
- Thin controllers - delegate to Application layer
- Platform-specific concerns only

```csharp
// ✅ GOOD: Thin controller
[HttpPost]
public async Task<IActionResult> CreateOrder(CreateOrderRequest request)
{
    var result = await _orderService.CreateOrderAsync(request.ToCommand());
    return result.IsSuccess ? Ok(result.Value) : BadRequest(result.Error);
}

// ❌ BAD: Business logic in controller
[HttpPost]
public async Task<IActionResult> CreateOrder(CreateOrderRequest request)
{
    if (request.Items.Sum(i => i.Price) < 10)
        return BadRequest("Minimum order is $10");
    // ... more business logic
}
```

### Application Layer

**Responsibility:** Use case orchestration and conditional business rules

**Contains:**
- Application Services (use case handlers)
- Commands and Queries (CQRS)
- DTOs for input/output
- **Conditional business rules** (context-dependent, policy-based)
- Cross-cutting concerns (authorization, validation, logging)
- Domain event handlers

**Rules:**
- Implements conditional/use-case specific business rules
- One use case = one unit of work (atomic)
- Publishes domain events
- Uses repository interfaces (not implementations)

```csharp
// ✅ GOOD: Conditional rules in Application Service
public async Task<Result<OrderDto>> CreateOrderAsync(CreateOrderCommand command)
{
    var customer = await _customerRepository.GetByIdAsync(command.CustomerId);
    if (customer is null) return Result.Fail("Customer not found");
    
    // Conditional rule: Check credit limit (depends on external data)
    var pendingTotal = await _orderRepository.GetPendingTotalAsync(customer.Id);
    if (pendingTotal + command.Total > customer.CreditLimit)
        return Result.Fail("Order exceeds credit limit");
    
    // Conditional rule: Discount policy (may vary by context)
    var discount = customer.IsLoyal ? 0.1m : 0m;
    
    // Conditional rule: Minimum order for online channel
    if (command.IsOnlineOrder && command.Total < 25)
        return Result.Fail("Minimum online order is $25");
    
    var order = Order.Create(customer, command.Items, discount);
    
    await _orderRepository.AddAsync(order);
    await _unitOfWork.SaveChangesAsync();
    
    return Result.Ok(order.ToDto());
}
```

### Domain Layer

**Responsibility:** Core business rules and invariants

**Contains:**
- Entities (identity + behavior)
- Value Objects (no identity, immutable)
- Aggregates (consistency boundaries)
- Domain Services (logic spanning entities within same aggregate)
- Domain Events
- Repository Interfaces (not implementations)
- Specifications
- **Core business rules** (invariants that must ALWAYS hold)

**Rules:**
- No dependencies on other layers
- No infrastructure concerns (no ORM attributes, no HTTP)
- Entities encapsulate behavior, not just data
- Aggregates enforce invariants

## Core vs Conditional Rules

| Rule Type | Layer | Characteristics | Examples |
|-----------|-------|-----------------|----------|
| **Core Rules** | Domain | Always true, no exceptions, entity protects itself | "Quantity > 0", "Email must be valid format", "Order must have lines" |
| **Conditional Rules** | Application | Context-dependent, policy-based, may need external data | "Min $25 for online orders", "Credit limit check", "Loyalty discount" |

**How to decide:**
- Can the entity enforce this alone? → **Domain**
- Does it need database queries? → **Application**
- Does it vary by use case/channel? → **Application**
- Would violation create impossible state? → **Domain**
- Is it a business policy that might change? → **Application**

```csharp
// ✅ GOOD: Entity with behavior
public class Order : Entity<OrderId>
{
    private readonly List<OrderLine> _lines = new();
    public IReadOnlyList<OrderLine> Lines => _lines.AsReadOnly();
    public Money Total => _lines.Sum(l => l.Subtotal);
    
    public static Order Create(Customer customer, IEnumerable<OrderLineDto> items)
    {
        var order = new Order(OrderId.New(), customer.Id);
        foreach (var item in items)
            order.AddLine(item.ProductId, item.Quantity, item.Price);
        return order;
    }
    
    public void AddLine(ProductId productId, int quantity, Money price)
    {
        if (quantity <= 0) throw new DomainException("Quantity must be positive");
        _lines.Add(new OrderLine(productId, quantity, price));
        AddDomainEvent(new OrderLineAddedEvent(Id, productId));
    }
}

// ❌ BAD: Anemic entity (data only)
public class Order
{
    public int Id { get; set; }
    public List<OrderLine> Lines { get; set; } = new();
    public decimal Total { get; set; }
}
```

### Infrastructure Layer

**Responsibility:** Technical implementations and external systems

**Contains:**
- Repository implementations
- Database context / ORM configuration
- External API clients
- Message queue implementations
- File system access
- Email/SMS services

**Rules:**
- Implements interfaces defined in Domain/Application
- Contains all platform-specific code
- "Get in, get out quick" - minimal logic

```csharp
// ✅ GOOD: Repository implementation
public class OrderRepository : IOrderRepository
{
    private readonly AppDbContext _context;
    
    public async Task<Order?> GetByIdAsync(OrderId id)
        => await _context.Orders
            .Include(o => o.Lines)
            .FirstOrDefaultAsync(o => o.Id == id);
    
    public async Task AddAsync(Order order)
        => await _context.Orders.AddAsync(order);
}
```

## DDD Building Blocks

### Quick Reference

| Building Block | Has Identity | Mutable | Purpose |
|----------------|--------------|---------|---------|
| **Entity** | Yes | Yes | Objects tracked by identity |
| **Value Object** | No | No | Descriptive, replaceable values |
| **Aggregate** | Yes | Yes | Consistency boundary with root entity |
| **Domain Service** | N/A | N/A | Logic not belonging to single entity |
| **Domain Event** | N/A | N/A | Something that happened in domain |
| **Repository** | N/A | N/A | Aggregate persistence abstraction |

### Entity vs Value Object

```csharp
// Entity: tracked by identity
public class Customer : Entity<CustomerId>
{
    public string Name { get; private set; }
    public Email Email { get; private set; }
}

// Value Object: tracked by value, immutable
public record Email
{
    public string Value { get; }
    
    public Email(string value)
    {
        if (!IsValid(value)) throw new DomainException("Invalid email");
        Value = value;
    }
    
    private static bool IsValid(string email) => /* validation */;
}
```

### Aggregates

**Rules:**
- One entity is the Aggregate Root
- External references only via root's ID
- Root enforces all invariants
- One aggregate = one transaction

```csharp
// Order is aggregate root
public class Order : AggregateRoot<OrderId>
{
    private readonly List<OrderLine> _lines = new();
    
    // Only root can modify children
    public void AddLine(ProductId productId, int quantity, Money price)
    {
        ValidateCanAddLine();
        _lines.Add(new OrderLine(productId, quantity, price));
    }
}

// ❌ BAD: External code modifying aggregate internals
order.Lines.Add(new OrderLine(...)); // Bypasses invariant checks
```

## Coding Principles

### DRY (Don't Repeat Yourself)

**Every piece of knowledge has a single, unambiguous representation.**

```csharp
// ❌ BAD: Duplicated validation
public void CreateUser(string email) { if (!email.Contains("@")) throw ...; }
public void UpdateEmail(string email) { if (!email.Contains("@")) throw ...; }

// ✅ GOOD: Single source of truth
public record Email(string Value)
{
    public Email(string value) : this(value)
    {
        if (!value.Contains("@")) throw new DomainException("Invalid email");
    }
}
```

### SOLID Principles

| Principle | Summary | Architectural Application |
|-----------|---------|--------------------------|
| **S**ingle Responsibility | One reason to change | Each layer has distinct responsibility |
| **O**pen/Closed | Open for extension, closed for modification | Use interfaces, strategy pattern |
| **L**iskov Substitution | Subtypes must be substitutable | Aggregates enforce invariants |
| **I**nterface Segregation | Small, focused interfaces | Repository per aggregate |
| **D**ependency Inversion | Depend on abstractions | Domain defines interfaces, Infra implements |

### Additional Principles

- **YAGNI** - Don't build it until you need it
- **KISS** - Keep solutions simple
- **Separation of Concerns** - Each module handles one thing
- **Explicit Dependencies** - No hidden service locators

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Business logic in controllers | Move to Domain layer |
| Anemic domain models (data bags) | Add behavior to entities |
| Repository returning DTOs | Return domain objects |
| Domain depending on ORM | Use POCO entities, configure in Infra |
| Aggregate referencing another aggregate | Use IDs only |
| Application layer with business rules | Move rules to Domain |
| Fat services, thin entities | Entities should have behavior |

## Layer Dependency Violations

```csharp
// ❌ Domain layer importing Infrastructure
using Microsoft.EntityFrameworkCore; // VIOLATION

// ❌ Domain layer importing Application
using MyApp.Application.DTOs; // VIOLATION

// ✅ Domain layer - only domain concerns
using MyApp.Domain.ValueObjects;
using MyApp.Domain.Events;
```

## Project Structure Example

```
src/
├── MyApp.Presentation/          # UI, Controllers
│   ├── Controllers/
│   └── ViewModels/
├── MyApp.Application/           # Use Cases, CQRS
│   ├── Commands/
│   ├── Queries/
│   └── Services/
├── MyApp.Domain/                # Core Business Logic
│   ├── Entities/
│   ├── ValueObjects/
│   ├── Services/
│   ├── Events/
│   └── Repositories/            # Interfaces only
├── MyApp.Infrastructure/        # External Concerns
│   ├── Persistence/
│   ├── ExternalServices/
│   └── Messaging/
└── MyApp.Domain.Shared/         # Shared enums, constants
```

## Decision Guide

```
Where does this code belong?

Is it UI/display related?
  → Presentation Layer

Is it orchestrating a use case?
  → Application Layer

Is it a core business rule?
  → Domain Layer

Is it talking to external systems?
  → Infrastructure Layer
```
