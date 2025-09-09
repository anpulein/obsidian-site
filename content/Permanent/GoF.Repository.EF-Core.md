---
aliases:
  - GoF.Repository.EF-Core
  - EF Core
tags:
  - type/permanent
  - status/active
  - design-pattern/reposiroty
  - tech/csharp
  - tech/ef-core
  - tech/asp-net
  - concept/ddd
  - concept/clean-architecture
  - concept/dependency-inje
  - concept/dependency-injection
  - area/architecture
title: GoF.Repository.EF-Core
linter-yaml-title-alias: GoF.Repository.EF-Core
date created: Tuesday, September 9th 2025, 5:25:02 am
date modified: Tuesday, September 9th 2025, 5:29:16 am
---

## 🏷️ Tags

#type/permanent #status/active #design-pattern/reposiroty #tech/csharp #tech/ef-core #tech/asp-net #concept/ddd #concept/clean-architecture #concept/dependency-injection #area/architecture 

---

# GoF.Repository.EF-Core

> [!info] 📋 О заметке Паттерн Repository в контексте .NET и Entity Framework Core - от базовых принципов до продвинутых техник реализации

---

## 🎯 Чек-лист изучения

- [ ] Понимание сути паттерна Repository
- [ ] Базовая реализация с EF Core
- [ ] Generic Repository vs Specific Repository
- [ ] Интеграция с Unit of Work
- [ ] Best practices и anti-patterns
- [ ] Тестирование репозиториев
- [ ] Альтернативы в современном .NET

---

## 📖 Содержание

1. [Основы паттерна](https://claude.ai/chat/c2395d9b-ae87-409b-afb5-ee8e54e66094#%D0%BE%D1%81%D0%BD%D0%BE%D0%B2%D1%8B-%D0%BF%D0%B0%D1%82%D1%82%D0%B5%D1%80%D0%BD%D0%B0)
2. [Базовая реализация](https://claude.ai/chat/c2395d9b-ae87-409b-afb5-ee8e54e66094#%D0%B1%D0%B0%D0%B7%D0%BE%D0%B2%D0%B0%D1%8F-%D1%80%D0%B5%D0%B0%D0%BB%D0%B8%D0%B7%D0%B0%D1%86%D0%B8%D1%8F)
3. [Generic Repository](https://claude.ai/chat/c2395d9b-ae87-409b-afb5-ee8e54e66094#generic-repository)
4. [Specific Repository](https://claude.ai/chat/c2395d9b-ae87-409b-afb5-ee8e54e66094#specific-repository)
5. [Интеграция с UoW](https://claude.ai/chat/c2395d9b-ae87-409b-afb5-ee8e54e66094#%D0%B8%D0%BD%D1%82%D0%B5%D0%B3%D1%80%D0%B0%D1%86%D0%B8%D1%8F-%D1%81-unit-of-work)
6. [Best Practices](https://claude.ai/chat/c2395d9b-ae87-409b-afb5-ee8e54e66094#best-practices)
7. [Тестирование](https://claude.ai/chat/c2395d9b-ae87-409b-afb5-ee8e54e66094#%D1%82%D0%B5%D1%81%D1%82%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5)
8. [Современные альтернативы](https://claude.ai/chat/c2395d9b-ae87-409b-afb5-ee8e54e66094#%D1%81%D0%BE%D0%B2%D1%80%D0%B5%D0%BC%D0%B5%D0%BD%D0%BD%D1%8B%D0%B5-%D0%B0%D0%BB%D1%8C%D1%82%D0%B5%D1%80%D0%BD%D0%B0%D1%82%D0%B8%D0%B2%D1%8B)

---

## Основы паттерна

> [!abstract] Repository Pattern **Репозиторий** — паттерн, который инкапсулирует логику доступа к данным и обеспечивает единообразный интерфейс для работы с коллекцией объектов

### Ключевые принципы

|Принцип|Описание|
|---|---|
|**Абстракция**|Скрывает детали реализации доступа к данным|
|**Тестируемость**|Позволяет легко мокать слой данных|
|**Разделение ответственности**|Отделяет бизнес-логику от логики доступа к данным|
|**Единообразие**|Предоставляет консистентный API для CRUD операций|

### Место в архитектуре

```
┌─────────────────┐
│   Presentation  │
└─────────────────┘
         ↓
┌─────────────────┐
│   Application   │ ← Использует IRepository<T>
└─────────────────┘
         ↓
┌─────────────────┐
│   Infrastructure│ ← Реализует Repository с EF Core
└─────────────────┘
         ↓
┌─────────────────┐
│    Database     │
└─────────────────┘
```

---

## Базовая реализация

### Интерфейс репозитория

```csharp
public interface IRepository<T> where T : class
{
    Task<T?> GetByIdAsync(int id);
    Task<IEnumerable<T>> GetAllAsync();
    Task<IEnumerable<T>> FindAsync(Expression<Func<T, bool>> predicate);
    
    Task<T> AddAsync(T entity);
    Task<IEnumerable<T>> AddRangeAsync(IEnumerable<T> entities);
    
    void Update(T entity);
    void UpdateRange(IEnumerable<T> entities);
    
    void Remove(T entity);
    void RemoveRange(IEnumerable<T> entities);
    
    Task<int> CountAsync();
    Task<bool> ExistsAsync(int id);
}
```

### Базовая реализация с EF Core

```csharp
public class Repository<T> : IRepository<T> where T : class
{
    protected readonly DbContext _context;
    protected readonly DbSet<T> _dbSet;

    public Repository(DbContext context)
    {
        _context = context;
        _dbSet = context.Set<T>();
    }

    public virtual async Task<T?> GetByIdAsync(int id)
    {
        return await _dbSet.FindAsync(id);
    }

    public virtual async Task<IEnumerable<T>> GetAllAsync()
    {
        return await _dbSet.ToListAsync();
    }

    public virtual async Task<IEnumerable<T>> FindAsync(
        Expression<Func<T, bool>> predicate)
    {
        return await _dbSet.Where(predicate).ToListAsync();
    }

    public virtual async Task<T> AddAsync(T entity)
    {
        var entry = await _dbSet.AddAsync(entity);
        return entry.Entity;
    }

    public virtual async Task<IEnumerable<T>> AddRangeAsync(
        IEnumerable<T> entities)
    {
        await _dbSet.AddRangeAsync(entities);
        return entities;
    }

    public virtual void Update(T entity)
    {
        _dbSet.Update(entity);
    }

    public virtual void UpdateRange(IEnumerable<T> entities)
    {
        _dbSet.UpdateRange(entities);
    }

    public virtual void Remove(T entity)
    {
        _dbSet.Remove(entity);
    }

    public virtual void RemoveRange(IEnumerable<T> entities)
    {
        _dbSet.RemoveRange(entities);
    }

    public virtual async Task<int> CountAsync()
    {
        return await _dbSet.CountAsync();
    }

    public virtual async Task<bool> ExistsAsync(int id)
    {
        return await _dbSet.FindAsync(id) != null;
    }
}
```

---

## Generic Repository

> [!tip] Generic Repository Универсальная реализация репозитория, подходящая для простых CRUD операций

### Расширенная версия

```csharp
public class GenericRepository<T> : IRepository<T> where T : class
{
    private readonly DbContext _context;
    private readonly DbSet<T> _dbSet;

    public GenericRepository(DbContext context)
    {
        _context = context;
        _dbSet = context.Set<T>();
    }

    public virtual async Task<T?> GetByIdAsync(int id)
    {
        return await _dbSet.FindAsync(id);
    }

    public virtual async Task<IEnumerable<T>> GetAllAsync(
        Expression<Func<T, bool>>? filter = null,
        Func<IQueryable<T>, IOrderedQueryable<T>>? orderBy = null,
        string includeProperties = "")
    {
        IQueryable<T> query = _dbSet;

        if (filter != null)
        {
            query = query.Where(filter);
        }

        foreach (var includeProperty in includeProperties
            .Split(new[] { ',' }, StringSplitOptions.RemoveEmptyEntries))
        {
            query = query.Include(includeProperty);
        }

        if (orderBy != null)
        {
            return await orderBy(query).ToListAsync();
        }

        return await query.ToListAsync();
    }

    public virtual async Task<PagedResult<T>> GetPagedAsync(
        int page, 
        int pageSize,
        Expression<Func<T, bool>>? filter = null,
        Func<IQueryable<T>, IOrderedQueryable<T>>? orderBy = null)
    {
        IQueryable<T> query = _dbSet;

        if (filter != null)
        {
            query = query.Where(filter);
        }

        var totalCount = await query.CountAsync();

        if (orderBy != null)
        {
            query = orderBy(query);
        }

        var items = await query
            .Skip((page - 1) * pageSize)
            .Take(pageSize)
            .ToListAsync();

        return new PagedResult<T>
        {
            Items = items,
            TotalCount = totalCount,
            Page = page,
            PageSize = pageSize
        };
    }
}
```

### Регистрация в DI

```csharp
// Program.cs или Startup.cs
services.AddScoped(typeof(IRepository<>), typeof(GenericRepository<>));
```

---

## Specific Repository

> [!warning] Специфичные репозитории Для сложной бизнес-логики создавайте специализированные репозитории

### Интерфейс для конкретной сущности

```csharp
public interface IUserRepository : IRepository<User>
{
    Task<User?> GetByEmailAsync(string email);
    Task<IEnumerable<User>> GetActiveUsersAsync();
    Task<IEnumerable<User>> GetUsersByRoleAsync(UserRole role);
    Task<bool> IsEmailUniqueAsync(string email, int? excludeUserId = null);
    Task<UserStatistics> GetUserStatisticsAsync(int userId);
}
```

### Реализация специфичного репозитория

```csharp
public class UserRepository : Repository<User>, IUserRepository
{
    public UserRepository(ApplicationDbContext context) : base(context)
    {
    }

    public async Task<User?> GetByEmailAsync(string email)
    {
        return await _dbSet
            .Include(u => u.Profile)
            .Include(u => u.Roles)
            .FirstOrDefaultAsync(u => u.Email == email);
    }

    public async Task<IEnumerable<User>> GetActiveUsersAsync()
    {
        return await _dbSet
            .Where(u => u.IsActive && !u.IsDeleted)
            .OrderBy(u => u.LastName)
            .ThenBy(u => u.FirstName)
            .ToListAsync();
    }

    public async Task<IEnumerable<User>> GetUsersByRoleAsync(UserRole role)
    {
        return await _dbSet
            .Include(u => u.Roles)
            .Where(u => u.Roles.Any(r => r.Name == role.ToString()))
            .ToListAsync();
    }

    public async Task<bool> IsEmailUniqueAsync(string email, int? excludeUserId = null)
    {
        var query = _dbSet.Where(u => u.Email == email);
        
        if (excludeUserId.HasValue)
        {
            query = query.Where(u => u.Id != excludeUserId.Value);
        }

        return !await query.AnyAsync();
    }

    public async Task<UserStatistics> GetUserStatisticsAsync(int userId)
    {
        // Сложная логика получения статистики
        var user = await _dbSet
            .Include(u => u.Orders)
            .Include(u => u.Reviews)
            .FirstOrDefaultAsync(u => u.Id == userId);

        if (user == null) return new UserStatistics();

        return new UserStatistics
        {
            TotalOrders = user.Orders.Count,
            TotalSpent = user.Orders.Sum(o => o.TotalAmount),
            AverageOrderValue = user.Orders.Any() ? 
                user.Orders.Average(o => o.TotalAmount) : 0,
            ReviewsCount = user.Reviews.Count
        };
    }
}
```

---

## Интеграция с Unit of Work

> [!note] Связь с UoW Repository часто используется вместе с [[Unity of Work|Unity of Work]] для управления транзакциями

### Интерфейс UoW

```csharp
public interface IUnitOfWork : IDisposable
{
    IUserRepository Users { get; }
    IOrderRepository Orders { get; }
    IProductRepository Products { get; }
    
    Task<int> SaveChangesAsync();
    Task BeginTransactionAsync();
    Task CommitTransactionAsync();
    Task RollbackTransactionAsync();
}
```

### Реализация UoW

```csharp
public class UnitOfWork : IUnitOfWork
{
    private readonly ApplicationDbContext _context;
    private IDbContextTransaction? _transaction;

    private IUserRepository? _users;
    private IOrderRepository? _orders;
    private IProductRepository? _products;

    public UnitOfWork(ApplicationDbContext context)
    {
        _context = context;
    }

    public IUserRepository Users => 
        _users ??= new UserRepository(_context);

    public IOrderRepository Orders => 
        _orders ??= new OrderRepository(_context);

    public IProductRepository Products => 
        _products ??= new ProductRepository(_context);

    public async Task<int> SaveChangesAsync()
    {
        return await _context.SaveChangesAsync();
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
        _context.Dispose();
    }
}
```

---

## Best Practices

> [!success] Рекомендации Следуй этим принципам для эффективного использования Repository Pattern

### ✅ Что делать

|Практика|Описание|
|---|---|
|**Async методы**|Используй асинхронные операции для I/O|
|**Специфичные репозитории**|Создавай для сложной бизнес-логики|
|**Include стратегия**|Продумывай загрузку связанных данных|
|**Валидация параметров**|Проверяй входные параметры|
|**Логирование**|Добавляй логи для отладки|

```csharp
public class UserRepository : Repository<User>, IUserRepository
{
    private readonly ILogger<UserRepository> _logger;

    public UserRepository(
        ApplicationDbContext context, 
        ILogger<UserRepository> logger) : base(context)
    {
        _logger = logger;
    }

    public async Task<User?> GetByEmailAsync(string email)
    {
        if (string.IsNullOrWhiteSpace(email))
        {
            throw new ArgumentException("Email cannot be empty", nameof(email));
        }

        _logger.LogInformation("Searching user by email: {Email}", email);

        try
        {
            var user = await _dbSet
                .Include(u => u.Profile)
                .FirstOrDefaultAsync(u => u.Email == email);

            _logger.LogInformation("User {Found} by email: {Email}", 
                user != null ? "found" : "not found", email);

            return user;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error searching user by email: {Email}", email);
            throw;
        }
    }
}
```

### ❌ Anti-patterns

> [!danger] Чего избегать
> 
> - **Repository поверх Repository** - не используй Repository поверх EF Core без необходимости
> - **Слишком generic** - не создавай универсальный репозиторий для всего
> - **Игнорирование IQueryable** - не преобразовывай в List раньше времени
> - **Отсутствие using/disposal** - не забывай про освобождение ресурсов

---

## Тестирование

### Unit тесты для репозитория

```csharp
public class UserRepositoryTests
{
    private ApplicationDbContext GetInMemoryContext()
    {
        var options = new DbContextOptionsBuilder<ApplicationDbContext>()
            .UseInMemoryDatabase(databaseName: Guid.NewGuid().ToString())
            .Options;

        return new ApplicationDbContext(options);
    }

    [Fact]
    public async Task GetByEmailAsync_ExistingEmail_ReturnsUser()
    {
        // Arrange
        await using var context = GetInMemoryContext();
        var repository = new UserRepository(context);
        
        var user = new User 
        { 
            Email = "test@example.com", 
            FirstName = "John", 
            LastName = "Doe" 
        };
        
        context.Users.Add(user);
        await context.SaveChangesAsync();

        // Act
        var result = await repository.GetByEmailAsync("test@example.com");

        // Assert
        Assert.NotNull(result);
        Assert.Equal("test@example.com", result.Email);
        Assert.Equal("John", result.FirstName);
    }

    [Fact]
    public async Task GetByEmailAsync_NonExistingEmail_ReturnsNull()
    {
        // Arrange
        await using var context = GetInMemoryContext();
        var repository = new UserRepository(context);

        // Act
        var result = await repository.GetByEmailAsync("notfound@example.com");

        // Assert
        Assert.Null(result);
    }
}
```

### Мокирование для тестов сервисов

```csharp
public class UserServiceTests
{
    [Fact]
    public async Task CreateUserAsync_UniqueEmail_CreatesUser()
    {
        // Arrange
        var mockRepository = new Mock<IUserRepository>();
        var mockUoW = new Mock<IUnitOfWork>();
        
        mockRepository
            .Setup(r => r.IsEmailUniqueAsync("test@example.com", null))
            .ReturnsAsync(true);
            
        mockUoW.Setup(u => u.Users).Returns(mockRepository.Object);
        mockUoW.Setup(u => u.SaveChangesAsync()).ReturnsAsync(1);

        var service = new UserService(mockUoW.Object);

        // Act
        var result = await service.CreateUserAsync(new CreateUserRequest
        {
            Email = "test@example.com",
            FirstName = "John",
            LastName = "Doe"
        });

        // Assert
        Assert.True(result.Success);
        mockRepository.Verify(r => r.AddAsync(It.IsAny<User>()), Times.Once);
        mockUoW.Verify(u => u.SaveChangesAsync(), Times.Once);
    }
}
```

---

## Современные альтернативы

> [!info] Альтернативы Repository Pattern В современном .NET есть несколько подходов, которые могут заменить классический Repository Pattern

### 1. Прямое использование EF Core

```csharp
public class UserService
{
    private readonly ApplicationDbContext _context;

    public UserService(ApplicationDbContext context)
    {
        _context = context;
    }

    public async Task<User?> GetUserByEmailAsync(string email)
    {
        return await _context.Users
            .Include(u => u.Profile)
            .FirstOrDefaultAsync(u => u.Email == email);
    }
}
```

**Плюсы:**

- Прямой доступ к возможностям EF Core
- Меньше абстракций
- Лучшая производительность

**Минусы:**

- Сложнее тестировать
- Нарушение принципа инверсии зависимостей

### 2. MediatR + CQRS

```csharp
// Query
public record GetUserByEmailQuery(string Email) : IRequest<User?>;

// Handler
public class GetUserByEmailHandler : IRequestHandler<GetUserByEmailQuery, User?>
{
    private readonly ApplicationDbContext _context;

    public GetUserByEmailHandler(ApplicationDbContext context)
    {
        _context = context;
    }

    public async Task<User?> Handle(
        GetUserByEmailQuery request, 
        CancellationToken cancellationToken)
    {
        return await _context.Users
            .Include(u => u.Profile)
            .FirstOrDefaultAsync(u => u.Email == request.Email, cancellationToken);
    }
}

// Использование в контроллере
[ApiController]
public class UsersController : ControllerBase
{
    private readonly IMediator _mediator;

    public UsersController(IMediator mediator)
    {
        _mediator = mediator;
    }

    [HttpGet("{email}")]
    public async Task<ActionResult<User>> GetByEmail(string email)
    {
        var user = await _mediator.Send(new GetUserByEmailQuery(email));
        return user != null ? Ok(user) : NotFound();
    }
}
```

### 3. Minimal API с EF Core

```csharp
app.MapGet("/users/{email}", async (string email, ApplicationDbContext context) =>
{
    var user = await context.Users
        .Include(u => u.Profile)
        .FirstOrDefaultAsync(u => u.Email == email);
        
    return user != null ? Results.Ok(user) : Results.NotFound();
});
```

---

## 📝 Заключение

Repository Pattern остается полезным паттерном в .NET, особенно для:

- **Сложных доменов** с богатой бизнес-логикой
- **Тестируемого кода** с четким разделением слоев
- **Легаси проектов** где уже используется этот подход

Однако в современных приложениях часто достаточно прямого использования EF Core с [[MediatR|MediatR]] или [[Minimal Api|Minimal API]].

---

## 🔗 Связанные заметки

- [[Unity of Work|Unity of Work]] - управление транзакциями
- [[Dependency Injection|Dependency Injection]] - внедрение зависимостей
- [[MOC - Clean Architcture|Clean Architecture]] - архитектурный контекст
- [[Entity Framework Core|Entity Framework Core]] - ORM для .NET
- [[MediatR|MediatR]] - медиатор для CQRS
- [[Specification Pattern|Specification Pattern]] - для сложных запросов

---

_Последнее обновление: Tuesday, September 09, 2025_
