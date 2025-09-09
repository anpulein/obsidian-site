---
aliases:
  - GoF.Repository.UoW
  - Unit of Work
  - UoW
tags:
  - type/permanent
  - status/active
  - area/architecture
  - design-pattern/uow
  - design-pattern/reposiroty
  - tech/csharp
  - tech/asp-net
  - tech/ef-core
  - concept/ddd
  - concept/clean-architecture
title: GoF.Repository.UoW
linter-yaml-title-alias: GoF.Repository.UoW
date created: Tuesday, September 9th 2025, 5:43:18 am
date modified: Tuesday, September 9th 2025, 5:46:05 am
---

## 🏷️ Tags

#type/permanent #status/active #area/architecture #design-pattern/uow #design-pattern/reposiroty #tech/csharp #tech/asp-net #tech/ef-core #concept/ddd #concept/clean-architecture 

---

# GoF.Repository.UoW

> [!abstract] 📋 Что изучаем
> 
> - [ ] Определение и назначение Unit of Work
> - [ ] Проблемы, которые решает паттерн
> - [ ] Классическая реализация в .NET
> - [ ] Интеграция с Repository Pattern
> - [ ] EF Core как встроенный Unit of Work
> - [ ] Современные альтернативы

---

## 🎯 Определение

**Unit of Work Pattern** — поведенческий паттерн, который поддерживает список объектов, затронутых бизнес-транзакцией, и координирует запись изменений в базу данных.

> [!tip] 💡 Ключевая идея Собрать все изменения в рамках одной бизнес-операции и применить их атомарно в одной транзакции.

### Основные принципы

- **Атомарность** — все изменения применяются или откатываются вместе
- **Консистентность** — гарантия целостности данных
- **Координация** — управление несколькими репозиториями в одной транзакции
- **Оптимизация** — группировка изменений для эффективности

---

## 🔧 Оглавление

1. [Проблемы без Unit of Work](https://claude.ai/chat/368e05a3-a19a-4dcb-be8c-614458fe88cb#%D0%BF%D1%80%D0%BE%D0%B1%D0%BB%D0%B5%D0%BC%D1%8B-%D0%B1%D0%B5%D0%B7-unit-of-work)
2. [Классическая реализация](https://claude.ai/chat/368e05a3-a19a-4dcb-be8c-614458fe88cb#%D0%BA%D0%BB%D0%B0%D1%81%D1%81%D0%B8%D1%87%D0%B5%D1%81%D0%BA%D0%B0%D1%8F-%D1%80%D0%B5%D0%B0%D0%BB%D0%B8%D0%B7%D0%B0%D1%86%D0%B8%D1%8F)
3. [Интеграция с Repository](https://claude.ai/chat/368e05a3-a19a-4dcb-be8c-614458fe88cb#%D0%B8%D0%BD%D1%82%D0%B5%D0%B3%D1%80%D0%B0%D1%86%D0%B8%D1%8F-%D1%81-repository)
4. [EF Core как UoW](https://claude.ai/chat/368e05a3-a19a-4dcb-be8c-614458fe88cb#ef-core-%D0%BA%D0%B0%D0%BA-uow)
5. [Современные подходы](https://claude.ai/chat/368e05a3-a19a-4dcb-be8c-614458fe88cb#%D1%81%D0%BE%D0%B2%D1%80%D0%B5%D0%BC%D0%B5%D0%BD%D0%BD%D1%8B%D0%B5-%D0%BF%D0%BE%D0%B4%D1%85%D0%BE%D0%B4%D1%8B)
6. [Сравнение решений](https://claude.ai/chat/368e05a3-a19a-4dcb-be8c-614458fe88cb#%D1%81%D1%80%D0%B0%D0%B2%D0%BD%D0%B5%D0%BD%D0%B8%D0%B5-%D1%80%D0%B5%D1%88%D0%B5%D0%BD%D0%B8%D0%B9)

---

## 🚨 Проблемы без Unit of Work

### Проблема координации репозиториев

```csharp
// ❌ Проблема: изменения в разных репозиториях не синхронизированы
public async Task TransferMoney(int fromAccountId, int toAccountId, decimal amount)
{
    var fromAccount = await _accountRepository.GetByIdAsync(fromAccountId);
    var toAccount = await _accountRepository.GetByIdAsync(toAccountId);
    
    fromAccount.Withdraw(amount);
    toAccount.Deposit(amount);
    
    await _accountRepository.UpdateAsync(fromAccount);  // Может успеть
    await _accountRepository.UpdateAsync(toAccount);    // Может упасть
    
    // Результат: деньги списались, но не зачислились!
}
```

### Проблема множественных транзакций

```csharp
// ❌ Каждый репозиторий создает свою транзакцию
public class OrderService
{
    public async Task CreateOrderAsync(CreateOrderRequest request)
    {
        var order = new Order(request);
        await _orderRepository.AddAsync(order);        // Транзакция 1
        
        await _inventoryRepository.ReserveItemsAsync(  // Транзакция 2
            order.Items);
            
        await _emailRepository.QueueNotificationAsync( // Транзакция 3
            order.CustomerId);
            
        // Что если одна из операций упадет?
    }
}
```

---

## 🏗️ Классическая реализация

### Интерфейс Unit of Work

```csharp
public interface IUnitOfWork : IDisposable
{
    // Доступ к репозиториям
    IOrderRepository Orders { get; }
    ICustomerRepository Customers { get; }
    IProductRepository Products { get; }
    
    // Управление транзакцией
    Task<int> SaveChangesAsync(CancellationToken cancellationToken = default);
    Task BeginTransactionAsync();
    Task CommitTransactionAsync();
    Task RollbackTransactionAsync();
}
```

### Реализация с EF Core

```csharp
public class UnitOfWork : IUnitOfWork
{
    private readonly ApplicationDbContext _context;
    private IDbContextTransaction? _transaction;
    
    // Lazy-инициализация репозиториев
    private IOrderRepository? _orders;
    private ICustomerRepository? _customers;
    private IProductRepository? _products;
    
    public UnitOfWork(ApplicationDbContext context)
    {
        _context = context;
    }
    
    // Свойства репозиториев
    public IOrderRepository Orders => 
        _orders ??= new OrderRepository(_context);
        
    public ICustomerRepository Customers => 
        _customers ??= new CustomerRepository(_context);
        
    public IProductRepository Products => 
        _products ??= new ProductRepository(_context);
    
    // Управление транзакцией
    public async Task<int> SaveChangesAsync(CancellationToken cancellationToken = default)
    {
        return await _context.SaveChangesAsync(cancellationToken);
    }
    
    public async Task BeginTransactionAsync()
    {
        _transaction = await _context.Database.BeginTransactionAsync();
    }
    
    public async Task CommitTransactionAsync()
    {
        if (_transaction != null)
        {
            await _transaction.CommitAsync();
            await _transaction.DisposeAsync();
            _transaction = null;
        }
    }
    
    public async Task RollbackTransactionAsync()
    {
        if (_transaction != null)
        {
            await _transaction.RollbackAsync();
            await _transaction.DisposeAsync();
            _transaction = null;
        }
    }
    
    public void Dispose()
    {
        _transaction?.Dispose();
    }
}
```

---

## 🤝 Интеграция с Repository

### Базовый репозиторий

```csharp
public abstract class Repository<T> where T : class
{
    protected readonly ApplicationDbContext Context;
    protected readonly DbSet<T> DbSet;
    
    protected Repository(ApplicationDbContext context)
    {
        Context = context;
        DbSet = context.Set<T>();
    }
    
    public virtual async Task<T?> GetByIdAsync(int id)
        => await DbSet.FindAsync(id);
    
    public virtual async Task<IEnumerable<T>> GetAllAsync()
        => await DbSet.ToListAsync();
    
    public virtual void Add(T entity)
        => DbSet.Add(entity);
    
    public virtual void Update(T entity)
        => DbSet.Update(entity);
    
    public virtual void Delete(T entity)
        => DbSet.Remove(entity);
}
```

### Конкретный репозиторий

```csharp
public class OrderRepository : Repository<Order>, IOrderRepository
{
    public OrderRepository(ApplicationDbContext context) : base(context) { }
    
    public async Task<Order?> GetOrderWithItemsAsync(int orderId)
    {
        return await DbSet
            .Include(o => o.Items)
            .FirstOrDefaultAsync(o => o.Id == orderId);
    }
    
    public async Task<IEnumerable<Order>> GetOrdersByCustomerAsync(int customerId)
    {
        return await DbSet
            .Where(o => o.CustomerId == customerId)
            .ToListAsync();
    }
}
```

### Использование в сервисе

```csharp
public class OrderService
{
    private readonly IUnitOfWork _unitOfWork;
    
    public OrderService(IUnitOfWork unitOfWork)
    {
        _unitOfWork = unitOfWork;
    }
    
    public async Task<Result> CreateOrderAsync(CreateOrderRequest request)
    {
        try
        {
            await _unitOfWork.BeginTransactionAsync();
            
            // Работаем с несколькими репозиториями
            var customer = await _unitOfWork.Customers.GetByIdAsync(request.CustomerId);
            if (customer == null) 
                return Result.Failure("Customer not found");
            
            var order = new Order(request.CustomerId, request.Items);
            _unitOfWork.Orders.Add(order);
            
            // Резервируем товары
            foreach (var item in request.Items)
            {
                var product = await _unitOfWork.Products.GetByIdAsync(item.ProductId);
                if (product?.Stock < item.Quantity)
                    return Result.Failure($"Insufficient stock for {product?.Name}");
                
                product.ReserveStock(item.Quantity);
                _unitOfWork.Products.Update(product);
            }
            
            // Сохраняем все изменения в одной транзакции
            await _unitOfWork.SaveChangesAsync();
            await _unitOfWork.CommitTransactionAsync();
            
            return Result.Success();
        }
        catch (Exception ex)
        {
            await _unitOfWork.RollbackTransactionAsync();
            return Result.Failure(ex.Message);
        }
    }
}
```

---

## 🚀 EF Core как UoW

> [!info] 💡 Важно понимать **DbContext в Entity Framework Core уже реализует паттерн Unit of Work!**

### DbContext как естественный UoW

```csharp
public class OrderService
{
    private readonly ApplicationDbContext _context;
    
    public OrderService(ApplicationDbContext context)
    {
        _context = context;
    }
    
    public async Task<Result> CreateOrderAsync(CreateOrderRequest request)
    {
        using var transaction = await _context.Database.BeginTransactionAsync();
        
        try
        {
            // Все изменения отслеживаются в одном контексте
            var customer = await _context.Customers.FindAsync(request.CustomerId);
            if (customer == null) 
                return Result.Failure("Customer not found");
            
            var order = new Order(request.CustomerId, request.Items);
            _context.Orders.Add(order);
            
            // Работаем с несколькими сущностями
            foreach (var item in request.Items)
            {
                var product = await _context.Products.FindAsync(item.ProductId);
                if (product?.Stock < item.Quantity)
                    return Result.Failure($"Insufficient stock");
                
                product.ReserveStock(item.Quantity);
                // EF Core автоматически отслеживает изменения
            }
            
            // Одна операция SaveChanges для всех изменений
            await _context.SaveChangesAsync();
            await transaction.CommitAsync();
            
            return Result.Success();
        }
        catch (Exception ex)
        {
            await transaction.RollbackAsync();
            return Result.Failure(ex.Message);
        }
    }
}
```

### Упрощенный подход через расширения

```csharp
public static class DbContextExtensions
{
    public static async Task<T> ExecuteInTransactionAsync<T>(
        this DbContext context, 
        Func<Task<T>> operation)
    {
        using var transaction = await context.Database.BeginTransactionAsync();
        try
        {
            var result = await operation();
            await context.SaveChangesAsync();
            await transaction.CommitAsync();
            return result;
        }
        catch
        {
            await transaction.RollbackAsync();
            throw;
        }
    }
}

// Использование
public async Task<Result> CreateOrderAsync(CreateOrderRequest request)
{
    return await _context.ExecuteInTransactionAsync(async () =>
    {
        // Бизнес-логика здесь
        var order = new Order(request);
        _context.Orders.Add(order);
        
        return Result.Success();
    });
}
```

---

## 🆕 Современные подходы

### 1. MediatR с транзакционными Behaviors

```csharp
public class TransactionBehavior<TRequest, TResponse> : IPipelineBehavior<TRequest, TResponse>
    where TRequest : IRequest<TResponse>
{
    private readonly ApplicationDbContext _context;
    
    public TransactionBehavior(ApplicationDbContext context)
    {
        _context = context;
    }
    
    public async Task<TResponse> Handle(
        TRequest request, 
        RequestHandlerDelegate<TResponse> next, 
        CancellationToken cancellationToken)
    {
        // Если команда помечена как транзакционная
        if (request is ITransactionalRequest)
        {
            using var transaction = await _context.Database.BeginTransactionAsync(cancellationToken);
            try
            {
                var response = await next();
                await _context.SaveChangesAsync(cancellationToken);
                await transaction.CommitAsync(cancellationToken);
                return response;
            }
            catch
            {
                await transaction.RollbackAsync(cancellationToken);
                throw;
            }
        }
        
        return await next();
    }
}

// Использование
public class CreateOrderCommand : IRequest<Result>, ITransactionalRequest
{
    public int CustomerId { get; set; }
    public List<OrderItem> Items { get; set; }
}
```

### 2. Minimal APIs с транзакционными эндпоинтами

```csharp
public static class OrderEndpoints
{
    public static void MapOrderEndpoints(this IEndpointRouteBuilder app)
    {
        app.MapPost("/orders", CreateOrderAsync)
           .WithTransactionalScope(); // Кастомное расширение
    }
    
    private static async Task<IResult> CreateOrderAsync(
        CreateOrderRequest request,
        ApplicationDbContext context)
    {
        // DbContext автоматически управляет Unit of Work
        var order = new Order(request);
        context.Orders.Add(order);
        
        await context.SaveChangesAsync();
        return Results.Created($"/orders/{order.Id}", order);
    }
}

// Расширение для транзакционного scope
public static class TransactionalExtensions
{
    public static RouteHandlerBuilder WithTransactionalScope(this RouteHandlerBuilder builder)
    {
        return builder.AddEndpointFilter(async (context, next) =>
        {
            var dbContext = context.HttpContext.RequestServices.GetRequiredService<ApplicationDbContext>();
            
            using var transaction = await dbContext.Database.BeginTransactionAsync();
            try
            {
                var result = await next(context);
                await transaction.CommitAsync();
                return result;
            }
            catch
            {
                await transaction.RollbackAsync();
                throw;
            }
        });
    }
}
```

---

## 📊 Сравнение решений

|Подход|Плюсы|Минусы|Когда использовать|
|---|---|---|---|
|**Классический UoW**|Четкий контракт, тестируемость|Много boilerplate кода|Legacy системы, сложная логика координации|
|**DbContext как UoW**|Простота, встроенная поддержка|Привязка к EF Core|Большинство .NET приложений|
|**MediatR Behaviors**|Декларативность, чистота|Дополнительная зависимость|CQRS архитектура|
|**Транзакционные фильтры**|Минимум кода|Менее гибкий|Простые веб-API|

---

## ✅ Best Practices

> [!success] ✅ Рекомендации
> 
> - **Используй DbContext напрямую** в большинстве случаев
> - **Создавай явные транзакции** только когда нужен полный контроль
> - **Держи транзакции короткими** — минимизируй время блокировок
> - **Обрабатывай исключения** и всегда откатывай транзакции при ошибках
> - **Тестируй транзакционную логику** с реальной базой данных

> [!warning] ⚠️ Избегай
> 
> - **Длинных транзакций** — они блокируют ресурсы
> - **Nested transactions** без крайней необходимости
> - **Смешивания паттернов** — выбери один подход для проекта
> - **Излишней абстракции** — не создавай UoW если DbContext достаточно

---

## 🔗 Связанные концепции

- [[MOC - GoF - Repository]] — основной паттерн-компаньон
- [[EF Core - Change Tracking]] — механизм отслеживания изменений
- [[Clean Architecture - Application Layer]] — место для транзакционной логики
- [[MediatR - Pipeline Behaviors]] — декларативные транзакции
- [[Database Transactions]] — основы работы с транзакциями

---

## 🎯 Заключение

> [!example] 💡 Итог
> 
> **Unit of Work Pattern** решает важную задачу координации изменений в рамках бизнес-транзакции. В современном .NET:
> 
> - **DbContext уже реализует UoW** — используй его напрямую в 80% случаев
> - **Создавай явный UoW** только при работе с несколькими контекстами
> - **Выбирай современные подходы** — MediatR behaviors, транзакционные фильтры
> - **Помни о производительности** — короткие транзакции лучше длинных

Паттерн остается актуальным, но его реализация в .NET значительно упростилась благодаря возможностям Entity Framework Core.

