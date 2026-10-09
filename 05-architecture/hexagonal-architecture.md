# Hexagonal Architecture (Ports & Adapters)

> Hexagonal architecture organizes a service so that the **business domain is completely independent** of the surrounding technology.
> The database, the web framework — all are interchangeable details.
> What matters is the business logic, which lives at the center.

---

## The problem it solves

❌ Traditional layered architecture:
[HTTP Controller]
↓
[Service]
↓
[Repository]
↓
[Database]
Problem: The "Service" mixes business logic with framework calls.
If you change the framework, you break the business. If you want to test the business,
you need to simulate the database.

✓ Hexagonal Architecture:
[HTTP Controller]  [CLI]  [Test]  ← Primary Adapters (enter the hexagon)
│            │      │
└────────────┴──────┘
│
[Driving Port]  ← Interface that defines the domain's API
│
┌───────────────┐
│               │
│    DOMAIN     │  ← Pure business logic (C#), no external dependencies
│               │
└───────────────┘
│
[Driven Port]  ← Interface the domain needs from the outside world
│
┌────────────┴──────┐
│                   │
[EF Core Adapter]    [Image API Adapter]  ← Secondary Adapters (exit the hexagon)

---

## Folder structure

src/
├── domain/                          # The hexagon — pure C#, no frameworks, no external EF cores
│   ├── catalog/
│   │   ├── Product.cs               # Aggregate Root with invariants (stock >= 0)
│   │   └── Category.cs              # Reference entity (Seed rows)
│   ├── sales/
│   │   ├── Sale.cs                  # Aggregate Root with invariants (At-least-one-line check)
│   │   ├── SaleItem.cs              # Internal entity mapped inside Sale (Frozen values)
│   │   └── ports/
│   │       ├── in/
│   │       │   └── ICreateSaleUseCase.cs # Driving port: use case contract
│   │       └── out/
│   │           └── ISaleRepositoryPort.cs # Driven port: repository contract
│   └── shared/
│       └── value_objects/           # VOs shared between aggregates
│           ├── Money.cs             # Monomoneda amount validation ( numeric(18,2) )
│           └── Quantity.cs          # Quantity > 0 validation
│
├── application/                     # Use cases — orchestrate the domain in memory
│   └── sales/
│       ├── CreateSaleUseCase.cs     # Implements the driving port
│       └── dtos/
│           ├── CreateSaleRequest.cs
│           └── CreateSaleResponse.cs
│
├── infrastructure/                  # Everything external to the hexagon
│   ├── adapters/
│   │   ├── in/                      # Primary adapters — receive external API calls
│   │   │   └── http/
│   │   │       ├── SaleController.cs
│   │   │       └── SaleContracts.cs
│   │   └── out/                     # Secondary adapters — call the database engine
│   │       └── persistence/
│   │           ├── SaleRepositoryAdapter.cs # Implements the driven port
│   │           └── Configurations/          # EF Core Shadow properties and configurations
│   └── config/
│       └── DependencyInjection.cs   # IoC Container wiring
│
└── Program.cs                       # Bootstrap — connects adapters with ports

---

## The Ports

Ports are **interfaces** (abstract contracts). The domain defines them; adapters implement them.

### Driving Port (Input Port)

Defines what the domain can do — its public API from the outside's perspective.

```csharp
// src/domain/sales/ports/in/ICreateSaleUseCase.cs
namespace SimpleStockFlow.Domain.Sales.Ports.In;

public interface ICreateSaleUseCase
{
    Task<CreateSaleResponse> ExecuteAsync(CreateSaleRequest request);
}
```

### Driven Port (Output Port)

Defines what the domain needs from the outside world — without knowing how it is implemented.

```csharp
// src/domain/sales/ports/out/ISaleRepositoryPort.cs
namespace SimpleStockFlow.Domain.Sales.Ports.Out;

public interface ISaleRepositoryPort
{
    Task SaveAsync(Sale sale);
    Task<Sale?> FindByIdAsync(Guid id);
}
```

---

## The Adapters

### Primary Adapter — HTTP Controller

The HTTP controller translates the HTTP request to the domain use case.

```csharp
// src/infrastructure/adapters/in/http/SaleController.cs
using SimpleStockFlow.Domain.Sales.Ports.In;

[ApiController]
[Route("sales")]
public class SaleController : ControllerBase
{
    private readonly ICreateSaleUseCase _createSaleUseCase;

    public SaleController(ICreateSaleUseCase createSaleUseCase)
    {
        _createSaleUseCase = createSaleUseCase; // Inject the port, NOT implementation
    }

    [HttpPost]
    public async Task<IActionResult> Create([FromBody] CreateSaleHttpRequest body)
    {
        var request = new CreateSaleRequest(body.SoldByUsername, body.Items);
        var response = await _createSaleUseCase.ExecuteAsync(request);
        return Ok(response);
    }
}
```

### Secondary Adapter — Repository

The repository implements the driven port. The domain does not know PostgreSQL exists.

```csharp
// src/infrastructure/adapters/out/persistence/SaleRepositoryAdapter.cs
using SimpleStockFlow.Domain.Sales.Ports.Out;

public class SaleRepositoryAdapter : ISaleRepositoryPort
{
    private readonly SalesDbContext _context;

    public SaleRepositoryAdapter(SalesDbContext context)
    {
        _context = context;
    }

    public async Task SaveAsync(Sale sale)
    {
        // Translate Aggregate → qualified sales schema tables
        await _context.Set<Sale>().AddAsync(sale);
        await _context.SaveChangesAsync(); // Triggers constraints and monitors xmin token
    }

    public async Task<Sale?> FindByIdAsync(Guid id)
    {
        return await _context.Set<Sale>()
            .Include(s => s.Items)
            .FirstOrDefaultAsync(s => s.Id == id);
    }
}
```

---

## The Use Case (Application Service)

The use case orchestrates the domain. It uses driving and driven ports. It contains no business logic — that lives in the Aggregate.

```csharp
// src/application/sales/CreateSaleUseCase.cs
using SimpleStockFlow.Domain.Sales.Ports.In;
using SimpleStockFlow.Domain.Sales.Ports.Out;

public class CreateSaleUseCase : ICreateSaleUseCase
{
    private readonly ISaleRepositoryPort _saleRepository;

    public CreateSaleUseCase(ISaleRepositoryPort saleRepository)
    {
        _saleRepository = saleRepository;
    }

    public async Task<CreateSaleResponse> ExecuteAsync(CreateSaleRequest request)
    {
        // 1. Create the aggregate (business logic lives HERE, inside C# Domain)
        var sale = Sale.Create(Guid.NewGuid(), request.SoldByUsername);
        
        foreach (var item in request.Items)
        {
            sale.AddItem(item.ProductId, item.Quantity, item.UnitPrice, item.ProductName, item.CategoryName);
        }

        sale.EnsureConfirmable(); // Validates the aggregate in-memory invariants

        // 2. Persist through driving ports (isolated from framework infrastructure)
        await _saleRepository.SaveAsync(sale);

        return new CreateSaleResponse(sale.Id);
    }
}
```

---

## The Dependency Rule

> **Dependencies always point inward.**
> The domain does not import anything from application or infrastructure layers.
> Infrastructure imports from the domain (but never the other way around).

infrastructure/ → application/ → domain/
↑
CANNOT import anything from outside
