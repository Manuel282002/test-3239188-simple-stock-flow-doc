# Design Patterns and Microservices Guide

> This document is the project's pattern catalog.
> For each pattern: when to use it, when NOT to, and an implementation example.
> Patterns are not recipes — they are tools. Use them when the problem requires it.

---

## Index

**Design patterns (GoF and SOLID)**
1. [Creational patterns](#creational)
2. [Structural patterns](#structural)
3. [Behavioral patterns](#behavioral)

**Microservices patterns**
4. [System decomposition](#decomposition)
5. [Inter-service communication](#communication)
6. [Resilience](#resilience)
7. [Data and consistency](#data)
8. [Observability](#observability)

---

## Design patterns (GoF) {#creational}

### 1. Factory Method

**Problem:** You want to create objects without exposing the creation logic or coupling code to the concrete type.

**When to use it:**
- When the exact type of object to create is not known until runtime.
- When creation has complex logic (validations, formatting, and trimming states).

**Domain example:**

```csharp
// Factory Method — inside the Sale Aggregate Root
public class Sale 
{
    private readonly List<SaleItem> _items = new();
    public Guid Id { get; private set; }
    public string SoldByUsername { get; private set; }

    private Sale() { } // Required for EF Core reconstitution

    // Instead of new Sale(...), the domain enforces factory initialization
    public static Sale Create(Guid id, string username) 
    {
        if (string.IsNullOrWhiteSpace(username)) 
            throw new DomainException("Operator username cannot be empty.");
            
        return new Sale { Id = id, SoldByUsername = username.ToLower().Trim() };
    }
}
```

---

### 2. Builder

**Problem:** An object has many optional parameters and construction becomes unreadable.

**When to use it:** Complex configuration test objects or database seed data mock targets.

```csharp
// Builder — highly useful for isolated domain testing
var mockProduct = new ProductBuilder()
    .WithId(Guid.NewGuid())
    .WithName("Martillo de Uña 16oz")
    .WithPrice(new Money(150.50m))
    .WithStock(45)
    .WithCategory(Guid.Parse("11111111-1111-4111-8111-111111111111"))
    .Build();
```

---

### 3. Singleton (with caution)

**Problem:** A class must have exactly one instance.

**When to use it:** Database context connectivity pooling or environment variables records.

**WARNING:** Native Singletons make isolated unit testing difficult. Prefer dependency injection scoping.

```csharp
// ✓ Better: Singleton lifecycle state managed natively by the .NET IoC container
builder.Services.AddSingleton<IInfrastructureConfig, EnvironmentEnvConfig>();
```

---

### 4. Adapter (Structural pattern) {#structural}

**Problem:** You want to use an existing class but its interface does not match the one you need.

**When to use it:** Integration with external image binary infrastructure layers.

```csharp
// The domain hexagon defines the driven output port interface contract
public interface IExternalImageStoragePort 
{
    Task DeleteBinaryAsync(string imageKey);
}

// The secondary adapter translates the contract to the cloud storage API
public class CloudStorageAdapter : IExternalImageStoragePort 
{
    private readonly S3Client _client;

    public CloudStorageAdapter(S3Client client) => _client = client;

    public async Task DeleteBinaryAsync(string imageKey) 
    {
        // Translates domain key context into direct infrastructure removal calls
        await _client.DeleteObjectAsync("simple-stock-flow-bucket", imageKey);
    }
}
```

---

### 5. Decorator

**Problem:** You want to add behavior to an object without modifying it or inheriting from it.

**When to use it:** Auditing, tracking database transaction logging, or intercepting exceptions around use cases.

```csharp
// Concurrency handler decorator wrapping the driven repository port
public class ConcurrencyHandlingSaleRepositoryDecorator : ISaleRepositoryPort 
{
    private readonly ISaleRepositoryPort _innerRepository;

    public ConcurrencyHandlingSaleRepositoryDecorator(ISaleRepositoryPort innerRepository) 
    {
        _innerRepository = innerRepository;
    }

    public async Task SaveAsync(Sale sale) 
    {
        try 
        {
            await _innerRepository.SaveAsync(sale);
        }
        catch (DbUpdateConcurrencyException) 
        {
            // Intercepts structural concurrency conflicts from the xmin token state
            throw new DomainException("The stock state has mutated. Please retry transaction.");
        }
    }
}
```

---

### 6. Observer / Internal Event Bus {#behavioral}

**Problem:** An object needs to notify others without knowing them directly.

**When to use it:** To dispatch synchronous in-memory notifications across hexagon layers.

```csharp
// The Sale Aggregate root aggregates events — Use Cases execute synchronous dispatching
public class Sale 
{
    private readonly List<object> _domainEvents = new();
    public IReadOnlyCollection<object> DomainEvents => _domainEvents.AsReadOnly();

    public void AddItem(Guid productId, int qty, decimal price) 
    {
        // ... inner business logic validating limits ...
        _domainEvents.Push(new SaleItemAddedEvent(productId, qty));
    }
}
```

---

### 7. Strategy

**Problem:** You want to swap algorithms at runtime.

**When to use it:** Swapping report extraction formats or testing localized decimal rounding guards.

```csharp
public interface IRoundingStrategy 
{
    decimal Round(decimal value);
}

public class AwayFromZeroRounding : IRoundingStrategy 
{
    // Matches the Money VO business rule constraint enforced for the physical model
    public decimal Round(decimal value) => Math.Round(value, 2, MidpointRounding.AwayFromZero);
}
```

---

### 8. Template Method

**Problem:** An algorithm has a fixed structure but some steps vary.

**When to use it:** Enforcing fixed transaction steps for executing imports or exports.

```csharp
public abstract class DataSeedExporter 
{
    // Template Method — Fixed execution framework structure
    public async Task ExportSeedAsync() 
    {
        ValidateTargetSchema();
        await ExecuteInsertionAsync();
        LogCompletion();
    }

    protected abstract Task ExecuteInsertionAsync();
    protected virtual void ValidateTargetSchema() { /* Baseline checks */ }
    private void LogCompletion() { /* Structured JSON logging output */ }
}
```

---

## Microservices Patterns

### Decomposition {#decomposition}

#### API Gateway
- **Status:** `N/A` (Not Applicable)
- **Justification:** Discarded by core architecture design guidelines. Simple Stock Flow runs completely as a centralized monolithic C# web application architecture (`simple-stock-flow-api`). All requests terminate directly on the API routing endpoints layer, eliminating external gateway proxies or token translation layers.

#### Backend for Frontend (BFF)
- **Status:** `N/A`
- **Justification:** The frontend consumers (admin and seller operators) target unified payload structures. Mappings and data models are served through standard domain contracts from a single API context.

#### Strangler Fig
- **Status:** `N/A`
- **Justification:** The system has been singularized directly onto the unified `sales` schema from inception. There is no legacy or distributed microservice stack migration planned.

---

## Resilience, Consistency, and Observability Distributed Patterns
- **Status:** All distributed messaging patterns (Sagas, Outbox Pattern, Kafka/RabbitMQ brokers, Dead Letter Queues, and Circuit Breakers) are explicitly **omitted from the architecture footprint**.
- **Justification:** Transactions do not cross external distributed network paths. Data stability, inventory tracking safety, and race condition blocks are solved sychronously and atomically inside a single database transaction box, backed by PostgreSQL 16.14 native `NOT NULL` fields, `CHECK` constraints, and the automatic concurrency tracking system column `xmin` (ADR-002 / D-04).

