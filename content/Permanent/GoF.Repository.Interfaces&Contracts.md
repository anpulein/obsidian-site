---
aliases:
  - GoF.Repository.Interfaces&Contracts
  - Interfaces and Contracts
  - Интерфейсы и контракты
tags:
  - type/permanent
  - status/active
  - design-pattern/reposiroty
  - concept/repository
  - tech/csharp
  - tech/asp-net
  - concept/ddd
  - concept/clean-architecture
  - area/architecture
title: GoF.Repository.Interfaces&Contracts
linter-yaml-title-alias: GoF.Repository.Interfaces&Contracts
date created: Tuesday, September 9th 2025, 5:20:17 am
date modified: Tuesday, September 9th 2025, 5:30:19 am
---

## 🏷️ Tags

#type/permanent #status/active #design-pattern/reposiroty #concept/repository #tech/csharp #tech/asp-net #concept/ddd #concept/clean-architecture #area/architecture 

---

# GoF.Repository.Interfaces&Contracts

> [!info] 📋 Чек-лист изучения
> 
> - [ ] Понять назначение и принципы Repository Pattern
> - [ ] Изучить базовые интерфейсы репозитория
> - [ ] Освоить специализированные интерфейсы
> - [ ] Разобрать контракты и ограничения
> - [ ] Изучить интеграцию с DI контейнером
> - [ ] Применить на практических примерах

---

## 📑 Содержание

- [[#🎯 Назначение Repository Pattern]]
- [[#🔧 Базовые интерфейсы]]
- [[#🎨 Специализированные интерфейсы]]
- [[#📋 Контракты и ограничения]]
- [[#🔗 Интеграция с DI]]
- [[#💡 Практические примеры]]
- [[#🚨 Частые ошибки]]

---

## 🎯 Назначение Repository Pattern

> [!abstract] Определение **Repository Pattern** — паттерн проектирования, который инкапсулирует логику доступа к данным и предоставляет более объектно-ориентированный взгляд на слой персистентности.

### Ключевые принципы

|Принцип|Описание|
|---|---|
|**Абстракция**|Скрывает детали реализации доступа к данным|
|**Тестируемость**|Позволяет легко мокать для unit-тестов|
|**Централизация**|Концентрирует логику запросов в одном месте|
|**Разделение ответственности**|Отделяет бизнес-логику от логики доступа к данным|

---

## 🔧 Базовые интерфейсы

### Generic Repository Interface

```csharp
/// <summary>
/// Базовый интерфейс для generic репозитория
/// </summary>
/// <typeparam name="TEntity">Тип сущности</typeparam>
/// <typeparam name="TKey">Тип первичного ключа</typeparam>
public interface IRepository<TEntity, TKey>
    where TEntity : class
    where TKey : IEquatable<TKey>
{
    // Получение данных
    Task<TEntity?> GetByIdAsync(TKey id, CancellationToken cancellationToken = default);
    Task<IEnumerable<TEntity>> GetAllAsync(CancellationToken cancellationToken = default);
    Task<IEnumerable<TEntity>> FindAsync(Expression<Func<TEntity, bool>> predicate, CancellationToken cancellationToken = default);
    
    // Изменение данных
    Task<TEntity> AddAsync(TEntity entity, CancellationToken cancellationToken = default);
    Task<IEnumerable<TEntity>> AddRangeAsync(IEnumerable<TEntity> entities, CancellationToken cancellationToken = default);
    Task UpdateAsync(TEntity entity, CancellationToken cancellationToken = default);
    Task DeleteAsync(TKey id, CancellationToken cancellationToken = default);
    Task DeleteAsync(TEntity entity, CancellationToken cancellationToken = default);
    
    // Проверки существования
    Task<bool> ExistsAsync(TKey id, CancellationToken cancellationToken = default);
    Task<int> CountAsync(CancellationToken cancellationToken = default);
    Task<int> CountAsync(Expression<Func<TEntity, bool>> predicate, CancellationToken cancellationToken = default);
}
```

### Read-Only Repository

```csharp
/// <summary>
/// Интерфейс для read-only операций
/// Используется в CQRS для Query стороны
/// </summary>
public interface IReadOnlyRepository<TEntity, TKey>
    where TEntity : class
    where TKey : IEquatable<TKey>
{
    Task<TEntity?> GetByIdAsync(TKey id, CancellationToken cancellationToken = default);
    Task<IEnumerable<TEntity>> GetAllAsync(CancellationToken cancellationToken = default);
    Task<IEnumerable<TEntity>> FindAsync(Expression<Func<TEntity, bool>> predicate, CancellationToken cancellationToken = default);
    Task<bool> ExistsAsync(TKey id, CancellationToken cancellationToken = default);
    Task<int> CountAsync(CancellationToken cancellationToken = default);
}
```

---

## 🎨 Специализированные интерфейсы

### Repository с пагинацией

```csharp
/// <summary>
/// Модель для пагинированного результата
/// </summary>
public class PagedResult<T>
{
    public IReadOnlyList<T> Items { get; init; } = Array.Empty<T>();
    public int TotalCount { get; init; }
    public int PageNumber { get; init; }
    public int PageSize { get; init; }
    public int TotalPages => (int)Math.Ceiling((double)TotalCount / PageSize);
    public bool HasPreviousPage => PageNumber > 1;
    public bool HasNextPage => PageNumber < TotalPages;
}

/// <summary>
/// Интерфейс с поддержкой пагинации
/// </summary>
public interface IPagedRepository<TEntity, TKey> : IRepository<TEntity, TKey>
    where TEntity : class
    where TKey : IEquatable<TKey>
{
    Task<PagedResult<TEntity>> GetPagedAsync(
        int pageNumber, 
        int pageSize, 
        CancellationToken cancellationToken = default);
        
    Task<PagedResult<TEntity>> GetPagedAsync(
        Expression<Func<TEntity, bool>> predicate,
        int pageNumber, 
        int pageSize, 
        CancellationToken cancellationToken = default);
}
```

### Specification Pattern Integration

```csharp
/// <summary>
/// Базовая спецификация
/// </summary>
public abstract class Specification<T>
{
    public abstract Expression<Func<T, bool>> ToExpression();
    
    public bool IsSatisfiedBy(T entity)
    {
        var predicate = ToExpression().Compile();
        return predicate(entity);
    }
}

/// <summary>
/// Repository с поддержкой Specification Pattern
/// </summary>
public interface ISpecificationRepository<TEntity, TKey> : IRepository<TEntity, TKey>
    where TEntity : class
    where TKey : IEquatable<TKey>
{
    Task<IEnumerable<TEntity>> FindAsync(
        Specification<TEntity> specification, 
        CancellationToken cancellationToken = default);
        
    Task<TEntity?> FindSingleAsync(
        Specification<TEntity> specification, 
        CancellationToken cancellationToken = default);
        
    Task<int> CountAsync(
        Specification<TEntity> specification, 
        CancellationToken cancellationToken = default);
}
```

### Unit of Work Integration

```csharp
/// <summary>
/// Интерфейс Unit of Work
/// </summary>
public interface IUnitOfWork : IDisposable
{
    Task<int> SaveChangesAsync(CancellationToken cancellationToken = default);
    Task BeginTransactionAsync(CancellationToken cancellationToken = default);
    Task CommitTransactionAsync(CancellationToken cancellationToken = default);
    Task RollbackTransactionAsync(CancellationToken cancellationToken = default);
}

/// <summary>
/// Repository с поддержкой Unit of Work
/// </summary>
public interface IUnitOfWorkRepository<TEntity, TKey> : IRepository<TEntity, TKey>
    where TEntity : class
    where TKey : IEquatable<TKey>
{
    IUnitOfWork UnitOfWork { get; }
}
```

---

## 📋 Контракты и ограничения

### Базовые контракты

> [!warning] Важные контракты
> 
> - **Null Safety**: методы должны возвращать `null` или `default`, если сущность не найдена
> - **Thread Safety**: реализации должны быть потокобезопасными
> - **Transaction Scope**: операции должны выполняться в рамках транзакций
> - **Exception Handling**: специфичные исключения для различных сценариев

```csharp
/// <summary>
/// Исключения репозитория
/// </summary>
public class EntityNotFoundException : Exception
{
    public EntityNotFoundException(string entityName, object key) 
        : base($"Entity '{entityName}' with key '{key}' was not found.")
    {
    }
}

public class EntityAlreadyExistsException : Exception
{
    public EntityAlreadyExistsException(string entityName, object key) 
        : base($"Entity '{entityName}' with key '{key}' already exists.")
    {
    }
}

public class ConcurrencyException : Exception
{
    public ConcurrencyException(string message) : base(message)
    {
    }
}
```

### Ограничения типов

```csharp
/// <summary>
/// Базовая сущность с аудитом
/// </summary>
public abstract class AuditableEntity<TKey> where TKey : IEquatable<TKey>
{
    public TKey Id { get; set; } = default!;
    public DateTime CreatedAt { get; set; }
    public string CreatedBy { get; set; } = string.Empty;
    public DateTime? UpdatedAt { get; set; }
    public string? UpdatedBy { get; set; }
}

/// <summary>
/// Repository для аудируемых сущностей
/// </summary>
public interface IAuditableRepository<TEntity, TKey> : IRepository<TEntity, TKey>
    where TEntity : AuditableEntity<TKey>
    where TKey : IEquatable<TKey>
{
    Task<IEnumerable<TEntity>> GetByCreatedDateRangeAsync(
        DateTime startDate, 
        DateTime endDate, 
        CancellationToken cancellationToken = default);
        
    Task<IEnumerable<TEntity>> GetByCreatedByAsync(
        string createdBy, 
        CancellationToken cancellationToken = default);
}
```

---

## 🔗 Интеграция с DI

### Регистрация в контейнере

```csharp
// Program.cs или Startup.cs
public static class ServiceCollectionExtensions
{
    public static IServiceCollection AddRepositories(this IServiceCollection services)
    {
        // Generic repository
        services.AddScoped(typeof(IRepository<,>), typeof(Repository<,>));
        services.AddScoped(typeof(IReadOnlyRepository<,>), typeof(ReadOnlyRepository<,>));
        
        // Specialized repositories
        services.AddScoped<IUserRepository, UserRepository>();
        services.AddScoped<IProductRepository, ProductRepository>();
        
        // Unit of Work
        services.AddScoped<IUnitOfWork, UnitOfWork>();
        
        return services;
    }
}
```

### Конфигурация с опциями

```csharp
/// <summary>
/// Настройки репозитория
/// </summary>
public class RepositoryOptions
{
    public const string Section = "Repository";
    
    public int DefaultPageSize { get; set; } = 20;
    public int MaxPageSize { get; set; } = 100;
    public bool EnableCaching { get; set; } = true;
    public TimeSpan CacheExpiration { get; set; } = TimeSpan.FromMinutes(15);
}

// Регистрация
services.Configure<RepositoryOptions>(
    configuration.GetSection(RepositoryOptions.Section));
```

---

## 💡 Практические примеры

### Доменно-специфичный репозиторий

```csharp
/// <summary>
/// Репозиторий пользователей с доменной логикой
/// </summary>
public interface IUserRepository : IRepository<User, Guid>
{
    Task<User?> GetByEmailAsync(string email, CancellationToken cancellationToken = default);
    Task<IEnumerable<User>> GetActiveUsersAsync(CancellationToken cancellationToken = default);
    Task<IEnumerable<User>> GetUsersByRoleAsync(string role, CancellationToken cancellationToken = default);
    Task<bool> EmailExistsAsync(string email, CancellationToken cancellationToken = default);
}

/// <summary>
/// Реализация с EF Core
/// </summary>
public class UserRepository : Repository<User, Guid>, IUserRepository
{
    public UserRepository(ApplicationDbContext context) : base(context)
    {
    }

    public async Task<User?> GetByEmailAsync(string email, CancellationToken cancellationToken = default)
    {
        return await Context.Users
            .FirstOrDefaultAsync(u => u.Email == email, cancellationToken);
    }

    public async Task<IEnumerable<User>> GetActiveUsersAsync(CancellationToken cancellationToken = default)
    {
        return await Context.Users
            .Where(u => u.IsActive)
            .ToListAsync(cancellationToken);
    }

    public async Task<bool> EmailExistsAsync(string email, CancellationToken cancellationToken = default)
    {
        return await Context.Users
            .AnyAsync(u => u.Email == email, cancellationToken);
    }
}
```

### Использование в сервисе

```csharp
public class UserService
{
    private readonly IUserRepository _userRepository;
    private readonly IUnitOfWork _unitOfWork;

    public UserService(IUserRepository userRepository, IUnitOfWork unitOfWork)
    {
        _userRepository = userRepository;
        _unitOfWork = unitOfWork;
    }

    public async Task<User> CreateUserAsync(CreateUserRequest request)
    {
        if (await _userRepository.EmailExistsAsync(request.Email))
        {
            throw new EntityAlreadyExistsException(nameof(User), request.Email);
        }

        var user = new User
        {
            Email = request.Email,
            Name = request.Name,
            CreatedAt = DateTime.UtcNow
        };

        await _userRepository.AddAsync(user);
        await _unitOfWork.SaveChangesAsync();

        return user;
    }
}
```

---

## 🚨 Частые ошибки

> [!danger] Антипаттерны
> 
> **1. Leaky Abstraction**
> 
> ```csharp
> // ❌ Неправильно - утечка EF Core логики в интерфейс
> Task<IQueryable<User>> GetUsersQueryable();
> 
> // ✅ Правильно
> Task<IEnumerable<User>> GetUsersBySpecificationAsync(Specification<User> spec);
> ```
> 
> **2. God Repository**
> 
> ```csharp
> // ❌ Неправильно - слишком много методов
> interface IUserRepository
> {
>     Task<User> GetById(int id);
>     Task<User> GetByEmail(string email);
>     Task<IEnumerable<User>> GetByAge(int age);
>     // ... 50+ методов
> }
> ```
> 
> **3. Игнорирование Unit of Work**
> 
> ```csharp
> // ❌ Неправильно - каждый репозиторий сохраняет сам
> await userRepository.SaveAsync();
> await productRepository.SaveAsync(); // Две транзакции!
> 
> // ✅ Правильно
> await userRepository.AddAsync(user);
> await productRepository.AddAsync(product);
> await unitOfWork.SaveChangesAsync(); // Одна транзакция
> ```

---

## 📚 Связанные темы

- [[Unit of Work Pattern]] - паттерн для управления транзакциями
- [[Specification Pattern]] - для композиции сложных запросов
- [[MOC - ArchPat - CQRS|CQRS]] - разделение команд и запросов
- [[DDD.Aggregate|Aggregate]] - проектирование агрегатов
- [[EF Core Repository]] - конкретная реализация с Entity Framework
