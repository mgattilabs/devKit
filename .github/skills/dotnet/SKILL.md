# .NET / C# Skill — Neo Backend

> **Trigger**: `*.sln` o `*.csproj` trovato nella root del progetto
> **Stack**: C# 14 / .NET (ultima versione LTS)
> **Questa skill estende le regole generiche di Neo Backend con convenzioni .NET-specifiche.**

---

## Stack Obbligatorio

| Categoria       | Tecnologia                                     | Note                                    |
|-----------------|------------------------------------------------|-----------------------------------------|
| Linguaggio      | C# 14 / .NET (ultima versione LTS)             | Nullable enabled, ImplicitUsings enabled|
| API             | ASP.NET Core Minimal API                       | MAI Controller-based in progetti nuovi  |
| ORM             | Entity Framework Core (ultima versione)        | SQL Server o SQLite in dev              |
| CQRS            | **Manuale — ICommandHandler / IQueryHandler**  | ⚠️ MediatR è VIETATO                    |
| Validazione     | DataAnnotations o manuale                      | FluentValidation solo se già presente   |
| Error handling  | Result Pattern                                 | MAI eccezioni per business logic        |
| Testing         | xUnit + NSubstitute + FluentAssertions         | Un test per ogni handler                |
| DI              | Constructor injection con `IServiceCollection` | MAI Service Locator                     |

---

## ⚠️ MediatR è VIETATO (commerciale dalla v12+)

Non usare `IRequest<T>`, `IRequestHandler<T>`, `IMediator`, `ISender`. Pattern approvato: CQRS manuale.

---

## Architecture Invariants (NON MODIFICARE SENZA RICHIESTA ESPLICITA)

Valgono per **tutti** i progetti .NET. Qualsiasi piano che le viola è un errore.

### 1. Quattro progetti sempre, cartella radice `sources/`

```
sources/
├── [NomeProgetto].Api/              // Endpoint, middleware, composition root
├── [NomeProgetto].Application/      // Use cases, handler, abstrazioni
├── [NomeProgetto].Domain/           // Entità, value objects, eventi, interfacce
└── [NomeProgetto].Infrastructure/   // Persistenza, servizi esterni
```
Mai accorpare. Mai aggiungere un quinto progetto senza richiesta esplicita.

### 2. Api + Application: split per app context (Cms, Website, PWA, Planner, Identity)

```
sources/[NomeProgetto].Api/Endpoints/{AppContext}/{Feature}Endpoint.cs
sources/[NomeProgetto].Application/UseCases/{AppContext}/{Feature}/{Feature}Command.cs
                                                         {Feature}Handler.cs
                                                         {Feature}Query.cs
                                                         {Feature}QueryHandler.cs
                                                         {Feature}Response.cs
```

### 3. Domain + Infrastructure: split per feature di dominio

```
sources/[NomeProgetto].Domain/
├── Entities/{Feature}/   ├── ValueObjects/   ├── Events/   └── Interfaces/

sources/[NomeProgetto].Infrastructure/
├── Persistence/Database/
│   ├── AppDbContext.cs
│   └── Configurations/   // un file per aggregate/entity
├── Repositories/
└── Services/
```

### 4. EF Config: un file per aggregate/entity

Path obbligatorio: `sources/[NomeProgetto].Infrastructure/Persistence/Database/Configurations/`
Mai configurazioni inline nel DbContext. Sempre `IEntityTypeConfiguration<T>` in file separato.

**Dipendenze**: `Api → Infrastructure → Application → Domain`. Domain zero dipendenze.

---

## CQRS Manuale — Template Obbligatori

### Interfacce base (`[NomeProgetto].Application/Abstractions/`)

```csharp
// Tutte le write operations
public interface ICommandHandler<TCommand, TResult>
{
    Task<TResult> HandleAsync(TCommand command, CancellationToken ct);
}

// Tutte le read operations
public interface IQueryHandler<TQuery, TResult>
{
    Task<TResult> HandleAsync(TQuery query, CancellationToken ct);
}
```

### Command + Handler

```csharp
// Command — record, nessun comportamento
public record CreateOrderCommand(Guid CustomerId, List<OrderItemDto> Items);

// Handler — singola responsabilità, un solo metodo pubblico
public sealed class CreateOrderHandler(
    AppDbContext db,
    IOrderValidator validator)
    : ICommandHandler<CreateOrderCommand, Result<Guid>>
{
    public async Task<Result<Guid>> HandleAsync(
        CreateOrderCommand command, CancellationToken ct)
    {
        var validation = await validator.ValidateAsync(command, ct);
        if (!validation.IsValid)
            return Result<Guid>.Failure(string.Join(", ", validation.Errors));

        var order = Order.Create(command.CustomerId, command.Items);
        db.Orders.Add(order);
        await db.SaveChangesAsync(ct);

        return Result<Guid>.Success(order.Id);
    }
}
```

### Query + Handler

```csharp
// Query — record, nessun comportamento
public record GetOrderByIdQuery(Guid OrderId);

// Handler — legge con AsNoTracking + projection, mai entity completa
public sealed class GetOrderByIdQueryHandler(AppDbContext db)
    : IQueryHandler<GetOrderByIdQuery, Result<GetOrderByIdResponse>>
{
    public async Task<Result<GetOrderByIdResponse>> HandleAsync(
        GetOrderByIdQuery query, CancellationToken ct)
    {
        var response = await db.Orders
            .AsNoTracking()
            .Where(o => o.Id == query.OrderId)
            .Select(o => new GetOrderByIdResponse(
                o.Id, o.CustomerId, o.TotalAmount, o.Status, o.CreatedAt))
            .FirstOrDefaultAsync(ct);

        return response is null
            ? Result<GetOrderByIdResponse>.NotFound(query.OrderId)
            : Result<GetOrderByIdResponse>.Success(response);
    }
}
```

### Minimal API Endpoint

```csharp
// Endpoint — thin slice, zero business logic
public static class CreateOrderEndpoint
{
    public static void MapCreateOrder(this IEndpointRouteBuilder app)
    {
        app.MapPost("/api/orders", async (
            CreateOrderCommand command,
            ICommandHandler<CreateOrderCommand, Result<Guid>> handler,
            CancellationToken ct) =>
        {
            var result = await handler.HandleAsync(command, ct);

            return result.IsSuccess
                ? Results.Created($"/api/orders/{result.Value}", result.Value)
                : Results.BadRequest(result.Error);
        })
        .WithName("CreateOrder")
        .WithTags("Orders")
        .Produces<Guid>(201)
        .ProducesProblem(400)
        .RequireAuthorization();   // ← SEMPRE — nessun endpoint anonimo senza giustificazione
    }
}
```

### DI Registration — Feature Module

```csharp
// {Domain}Module.cs — ogni feature si registra da sola
public static class OrdersModule
{
    public static IServiceCollection AddOrders(this IServiceCollection services)
    {
        services.AddScoped<ICommandHandler<CreateOrderCommand, Result<Guid>>,
            CreateOrderHandler>();
        services.AddScoped<IQueryHandler<GetOrderByIdQuery, Result<GetOrderByIdResponse>>,
            GetOrderByIdQueryHandler>();
        return services;
    }

    public static IEndpointRouteBuilder MapOrders(this IEndpointRouteBuilder app)
    {
        app.MapCreateOrder();
        app.MapGetOrderById();
        return app;
    }
}

// Program.cs — composition root pulito
builder.Services.AddOrders();
app.MapOrders();
```

---

## Result Pattern — Template Obbligatorio

```csharp
// In [NomeProgetto].Application/Abstractions/Result.cs
public sealed record Result<T>
{
    public bool IsSuccess { get; private init; }
    public T? Value { get; private init; }
    public string? Error { get; private init; }

    private Result() { }

    public static Result<T> Success(T value) =>
        new() { IsSuccess = true, Value = value };

    public static Result<T> Failure(string error) =>
        new() { IsSuccess = false, Error = error };

    public static Result<T> NotFound(Guid id) =>
        new() { IsSuccess = false, Error = $"Entity with id '{id}' was not found." };
}

// Mapping Result → HTTP nei Minimal API Endpoints:
// IsSuccess              → 200 OK o 201 Created
// Failure (business)     → 400 BadRequest
// NotFound               → 404 NotFound
// Infrastructure failure → 500 (gestito dal GlobalExceptionHandler)
```

---

## Domain Model — Template DDD in C#

### Aggregate Root

```csharp
// Base class per tutti gli aggregate root
public abstract class AggregateRoot
{
    private readonly List<IDomainEvent> _domainEvents = [];
    public IReadOnlyCollection<IDomainEvent> DomainEvents => _domainEvents.AsReadOnly();

    protected void RaiseDomainEvent(IDomainEvent @event) => _domainEvents.Add(@event);
    public void ClearDomainEvents() => _domainEvents.Clear();
}

// Aggregate — costruttore privato, factory method, setter privati
public sealed class Order : AggregateRoot
{
    public OrderId Id { get; private init; }
    public CustomerId CustomerId { get; private init; }
    public OrderStatus Status { get; private set; }

    private readonly List<OrderItem> _items = [];
    public IReadOnlyCollection<OrderItem> Items => _items.AsReadOnly();

    private Order() { }   // EF Core richiede il costruttore senza parametri

    public static Order Create(CustomerId customerId, IEnumerable<OrderItem> items)
    {
        var itemList = items.ToList();
        if (itemList.Count == 0)
            throw new DomainRuleViolationException("L'ordine deve contenere almeno un articolo.");

        var order = new Order
        {
            Id = OrderId.New(),
            CustomerId = customerId,
            Status = OrderStatus.Draft,
        };

        foreach (var item in itemList)
            order._items.Add(item);

        order.RaiseDomainEvent(new OrderCreatedEvent(order.Id, customerId));
        return order;
    }

    public void Confirm()
    {
        if (Status != OrderStatus.Draft)
            throw new DomainRuleViolationException(
                $"Impossibile confermare un ordine in stato '{Status}'.");

        Status = OrderStatus.Confirmed;
        RaiseDomainEvent(new OrderConfirmedEvent(Id));
    }
}
```

### Strongly-Typed IDs

```csharp
// Ogni entity usa un ID tipizzato — mai Guid raw nel dominio
public readonly record struct OrderId(Guid Value)
{
    public static OrderId New() => new(Guid.NewGuid());
    public static OrderId From(Guid value)
    {
        if (value == Guid.Empty)
            throw new DomainException("OrderId non può essere vuoto.");
        return new OrderId(value);
    }
    public override string ToString() => Value.ToString();
}
```

### Value Objects

```csharp
// Record immutabili con factory method e validazione
public sealed record Money
{
    public decimal Amount { get; }
    public string Currency { get; }

    private Money(decimal amount, string currency)
    {
        Amount = amount;
        Currency = currency;
    }

    public static Money Create(decimal amount, string currency)
    {
        if (amount < 0)
            throw new DomainException("Il valore non può essere negativo.");
        if (string.IsNullOrWhiteSpace(currency) || currency.Length != 3)
            throw new DomainException("La valuta deve essere un codice ISO a 3 lettere.");
        return new Money(amount, currency.ToUpperInvariant());
    }

    public Money Add(Money other)
    {
        if (Currency != other.Currency)
            throw new DomainException($"Impossibile sommare {Currency} e {other.Currency}.");
        return Create(Amount + other.Amount, Currency);
    }
}
```

---

## EF Core — Regole Obbligatorie

Read: `AsNoTracking()` + `Select()` su DTO. Write: entità tracked. Mai caricare entità complete per read-only.

```csharp
// ✅ Read — AsNoTracking + projection
var summaries = await db.Orders
    .AsNoTracking()
    .Where(o => o.CustomerId == customerId)
    .Select(o => new OrderSummaryDto(o.Id, o.Status, o.CreatedAt))
    .ToListAsync(ct);

// ✅ Single entity read — FindAsync per primary key
var order = await db.Orders.FindAsync([id], ct);

// ✅ Write — entity tracked, SaveChangesAsync con CancellationToken
db.Orders.Add(order);
await db.SaveChangesAsync(ct);

// ❌ MAI .Result o .Wait()
// ❌ MAI SELECT * (caricare entità complete per letture)
// ❌ MAI lazy loading — usare Include() esplicito o projection
// ❌ MAI IQueryable esposto fuori dall'Infrastructure layer
```

### EF Configuration — un file per entity

```csharp
// sources/[NomeProgetto].Infrastructure/Persistence/Database/Configurations/OrderConfiguration.cs
public sealed class OrderConfiguration : IEntityTypeConfiguration<Order>
{
    public void Configure(EntityTypeBuilder<Order> builder)
    {
        builder.ToTable("Orders");

        // Strongly-typed ID
        builder.HasKey(o => o.Id);
        builder.Property(o => o.Id)
            .HasConversion(id => id.Value, value => OrderId.From(value));

        // Value Object come owned type
        builder.OwnsOne(o => o.TotalAmount, money =>
        {
            money.Property(m => m.Amount).HasColumnName("TotalAmount").HasPrecision(18, 2);
            money.Property(m => m.Currency).HasColumnName("Currency").HasMaxLength(3);
        });

        // Enum come stringa — mai come int
        builder.Property(o => o.Status)
            .HasConversion<string>()
            .HasMaxLength(50);

        // Indici per colonne frequentemente filtrate
        builder.HasIndex(o => o.CustomerId);
        builder.HasIndex(o => o.CreatedAt);
    }
}
```

### EF Migrations — MAI creare, solo suggerire

A fine implementazione aggiungere al riepilogo:
```
📦 MIGRAZIONE NECESSARIA
[elenco entity/configuration modificate]
dotnet ef migrations add [NomeDescrittivo] --project sources/[NomeProgetto].Infrastructure --startup-project sources/[NomeProgetto].Api
```
- MAI eseguire `dotnet ef migrations add` o `dotnet ef database update`
- Sempre nomi descrittivi (es. `AddOrderEntity`, `AddStageStartDate`)

---

## Testing — Template C#

La naming convention per i test è `MethodName_Condition_ExpectedResult()`.
La struttura è sempre Arrange → Act → Assert, senza commenti che lo dichiarano.

```csharp
public class CreateOrderHandlerTests
{
    private readonly AppDbContext _db;
    private readonly CreateOrderHandler _handler;

    public CreateOrderHandlerTests()
    {
        var options = new DbContextOptionsBuilder<AppDbContext>()
            .UseInMemoryDatabase(Guid.NewGuid().ToString())
            .Options;
        _db = new AppDbContext(options);
        _handler = new CreateOrderHandler(_db, Substitute.For<IOrderValidator>());
    }

    [Fact(DisplayName = "Ordine valido viene creato e restituisce Guid")]
    public async Task HandleAsync_ValidCommand_ReturnsSuccessWithGuid()
    {
        var command = new CreateOrderCommand(Guid.NewGuid(),
            [new OrderItemDto(Guid.NewGuid(), 2)]);

        var result = await _handler.HandleAsync(command, CancellationToken.None);

        result.IsSuccess.Should().BeTrue();
        result.Value.Should().NotBeEmpty();
    }

    [Fact(DisplayName = "Ordine senza articoli restituisce Failure")]
    public async Task HandleAsync_EmptyItems_ReturnsFailure()
    {
        var command = new CreateOrderCommand(Guid.NewGuid(), []);

        var result = await _handler.HandleAsync(command, CancellationToken.None);

        result.IsSuccess.Should().BeFalse();
        result.Error.Should().NotBeNullOrEmpty();
    }
}
```

---

## Naming Conventions C#

```
Classi e interfacce:  PascalCase            → UserService, IUserRepository
Metodi e proprietà:   PascalCase            → GetUserById, FirstName
Variabili locali:     camelCase             → userName, totalCount
Campi privati:        _camelCase            → _logger, _repository
Costanti:             PascalCase            → MaxRetryCount, SecondsInADay
Enumerazioni:         PascalCase (nome e valori) → UserStatus.Active
Record/DTO:           PascalCase + suffisso → CreateOrderCommand, OrderSummaryDto
Test methods:         MethodName_Condition_ExpectedResult
```

---

## Regole .NET-Specific

Queste regole si aggiungono alle regole assolute dell'agente generico:

1. **MAI MediatR** — `ICommandHandler` / `IQueryHandler` sempre
2. **MAI `.Result` o `.Wait()`** — sempre `async/await`
3. **MAI entity complete in query di lettura** — `AsNoTracking()` + `Select()`
4. **MAI Controller-based** in progetti nuovi — solo Minimal API
5. **MAI lazy loading** — usare `Include()` esplicito o projection
6. **MAI `IQueryable` esposto** fuori dall'Infrastructure layer
7. **Sempre** `CancellationToken ct` in ogni metodo async
8. **Sempre** `Nullable enable` nel `.csproj`
9. **Sempre** `ImplicitUsings enable` nel `.csproj`
10. **Sempre** enum persistiti come stringa, mai come int
11. **MAI creare migrazioni** — solo suggerire il comando a fine implementazione