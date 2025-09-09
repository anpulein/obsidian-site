---
aliases:
  - GoF.Repository.Generic
  - Generic
tags:
  - type/permanent
  - status/active
  - area/architecture
  - concept/repository
  - design-pattern/reposiroty
  - tech/csharp
  - tech/ef-core
  - impl/ef-core
  - concept/ddd
title: GoF.Repository.Generic
linter-yaml-title-alias: GoF.Repository.Generic
date created: Tuesday, September 9th 2025, 5:35:44 am
date modified: Tuesday, September 9th 2025, 5:51:37 am
---

## 🏷️ Tags

#type/permanent #status/active #area/architecture #concept/repository #design-pattern/reposiroty #tech/csharp #tech/ef-core #impl/ef-core #concept/ddd

---

# GoF.Repository.Generic

> [!info] 📋 О заметке Детальный разбор реализации Generic Repository Pattern в .NET с примерами кода, лучшими практиками и интеграцией с Entity Framework Core.

---

## 🎯 Что будет раскрыто

- [ ] Базовая реализация Generic Repository
- [ ] Расширенный интерфейс с фильтрацией и сортировкой
- [ ] Интеграция с EF Core и DbContext
- [ ] Async/await паттерны
- [ ] Обработка связанных сущностей (Include)
- [ ] Пагинация и спецификации
- [ ] Unit of Work integration

---

## 🏗️ Базовая реализация

### Интерфейс Generic Repository

```csharp
public interface IGenericRepository<T> where T : class
{
    // Чтение
    Task<T?> GetByIdAsync(int id);
    Task<T?> GetByIdAsync(int id, params Expression<Func<T, object>>[] includes);
    Task<IEnumerable<T>> GetAllAsync();
    Task<IEnumerable<T>> FindAsync(Expression<Func<T, bool>> predicate);
    Task<IEnumerable<T>> FindAsync(
        Expression<Func<T, bool>>? filter = null,
        Func<IQueryable<T>, IOrderedQueryable<T>>? orderBy = null,
        params Expression<Func<T, object>>[] includes);
    
    // Запись
    Task AddAsync(T entity);
    Task AddRangeAsync(IEnumerable<T> entities);
    void Update(T entity);
    void UpdateRange(IEnumerable<T> entities);
    void Remove(T entity);
    void RemoveRange(IEnumerable<T> entities);
    
    // Утилиты
    Task<bool> ExistsAsync(int id);
    Task<int> CountAsync(Expression<Func<T, bool>>? filter = null);
}
```

### Базовая реализация

```csharp
public class GenericRepository<T> : IGenericRepository<T> where T : class
{
    protected readonly DbContext _context;
    protected readonly DbSet<T> _dbSet;
    
    public GenericRepository(DbContext context)
    {
        _context = context ?? throw new ArgumentNullException(nameof(context));
        _dbSet = _context.Set<T>();
    }
    
    public virtual async Task<T?> GetByIdAsync(int id)
    {
        return await _dbSet.FindAsync(id);
    }
    
    public virtual async Task<T?> GetByIdAsync(int id, 
        params Expression<Func<T, object>>[] includes)
    {
        IQueryable<T> query = _dbSet;
        
        foreach (var include in includes)
        {
            query = query.Include(include);
        }
        
        // Предполагаем, что Id - это свойство
        var parameter = Expression.Parameter(typeof(T), "e");
        var property = Expression.Property(parameter, "Id");
        var constant = Expression.Constant(id);
        var equals = Expression.Equal(property, constant);
        var lambda = Expression.Lambda<Func<T, bool>>(equals, parameter);
        
        return await query.FirstOrDefaultAsync(lambda);
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
    
    public virtual async Task<IEnumerable<T>> FindAsync(
        Expression<Func<T, bool>>? filter = null,
        Func<IQueryable<T>, IOrderedQueryable<T>>? orderBy = null,
        params Expression<Func<T, object>>[] includes)
    {
        IQueryable<T> query = _dbSet;
        
        // Применяем фильтры
        if (filter != null)
        {
            query = query.Where(filter);
        }
        
        // Включаем связанные сущности
        foreach (var include in includes)
        {
            query = query.Include(include);
        }
        
        // Применяем сортировку
        if (orderBy != null)
        {
            query = orderBy(query);
        }
        
        return await query.ToListAsync();
    }
    
    public virtual async Task AddAsync(T entity)
    {
        await _dbSet.AddAsync(entity);
    }
    
    public virtual async Task AddRangeAsync(IEnumerable<T> entities)
    {
        await _dbSet.AddRangeAsync(entities);
    }
    
    public virtual void Update(T entity)
    {
        _dbSet.Attach(entity);
        _context.Entry(entity).State = EntityState.Modified;
    }
    
    public virtual void UpdateRange(IEnumerable<T> entities)
    {
        _dbSet.UpdateRange(entities);
    }
    
    public virtual void Remove(T entity)
    {
        if (_context.Entry(entity).State == EntityState.Detached)
        {
            _dbSet.Attach(entity);
        }
        _dbSet.Remove(entity);
    }
    
    public virtual void RemoveRange(IEnumerable<T> entities)
    {
        _dbSet.RemoveRange(entities);
    }
    
    public virtual async Task<bool> ExistsAsync(int id)
    {
        var entity = await _dbSet.FindAsync(id);
        return entity != null;
    }
    
    public virtual async Task<int> CountAsync(
        Expression<Func<T, bool>>? filter = null)
    {
        IQueryable<T> query = _dbSet;
        
        if (filter != null)
        {
            query = query.Where(filter);
        }
        
        return await query.CountAsync();
    }
}
```

---

## 🔄 Расширенная реализация с пагинацией

### Модель пагинации

```csharp
public class PagedResult<T>
{
    public IEnumerable<T> Data { get; set; } = new List<T>();
    public int TotalCount { get; set; }
    public int PageNumber { get; set; }
    public int PageSize { get; set; }
    public int TotalPages => (int)Math.Ceiling((double)TotalCount / PageSize);
    public bool HasPreviousPage => PageNumber > 1;
    public bool HasNextPage => PageNumber < TotalPages;
}

public class PagedRequest
{
    public int PageNumber { get; set; } = 1;
    public int PageSize { get; set; } = 10;
    
    public PagedRequest()
    {
        PageNumber = PageNumber < 1 ? 1 : PageNumber;
        PageSize = PageSize > 50 ? 50 : PageSize; // Ограничение
    }
}
```

### Расширенный интерфейс

```csharp
public interface IGenericRepositoryExtended<T> : IGenericRepository<T> where T : class
{
    Task<PagedResult<T>> GetPagedAsync(
        PagedRequest request,
        Expression<Func<T, bool>>? filter = null,
        Func<IQueryable<T>, IOrderedQueryable<T>>? orderBy = null,
        params Expression<Func<T, object>>[] includes);
        
    Task<PagedResult<TResult>> GetPagedAsync<TResult>(
        PagedRequest request,
        Expression<Func<T, TResult>> selector,
        Expression<Func<T, bool>>? filter = null,
        Func<IQueryable<T>, IOrderedQueryable<T>>? orderBy = null,
        params Expression<Func<T, object>>[] includes);
}
```

### Реализация с пагинацией

```csharp
public class GenericRepositoryExtended<T> : GenericRepository<T>, IGenericRepositoryExtended<T> 
    where T : class
{
    public GenericRepositoryExtended(DbContext context) : base(context) { }
    
    public virtual async Task<PagedResult<T>> GetPagedAsync(
        PagedRequest request,
        Expression<Func<T, bool>>? filter = null,
        Func<IQueryable<T>, IOrderedQueryable<T>>? orderBy = null,
        params Expression<Func<T, object>>[] includes)
    {
        IQueryable<T> query = _dbSet;
        
        // Применяем фильтры
        if (filter != null)
        {
            query = query.Where(filter);
        }
        
        // Подсчитываем общее количество до пагинации
        var totalCount = await query.CountAsync();
        
        // Включаем связанные сущности
        foreach (var include in includes)
        {
            query = query.Include(include);
        }
        
        // Применяем сортировку
        if (orderBy != null)
        {
            query = orderBy(query);
        }
        
        // Применяем пагинацию
        var data = await query
            .Skip((request.PageNumber - 1) * request.PageSize)
            .Take(request.PageSize)
            .ToListAsync();
        
        return new PagedResult<T>
        {
            Data = data,
            TotalCount = totalCount,
            PageNumber = request.PageNumber,
            PageSize = request.PageSize
        };
    }
    
    public virtual async Task<PagedResult<TResult>> GetPagedAsync<TResult>(
        PagedRequest request,
        Expression<Func<T, TResult>> selector,
        Expression<Func<T, bool>>? filter = null,
        Func<IQueryable<T>, IOrderedQueryable<T>>? orderBy = null,
        params Expression<Func<T, object>>[] includes)
    {
        IQueryable<T> query = _dbSet;
        
        if (filter != null)
        {
            query = query.Where(filter);
        }
        
        var totalCount = await query.CountAsync();
        
        foreach (var include in includes)
        {
            query = query.Include(include);
        }
        
        if (orderBy != null)
        {
            query = orderBy(query);
        }
        
        var data = await query
            .Skip((request.PageNumber - 1) * request.PageSize)
            .Take(request.PageSize)
            .Select(selector)
            .ToListAsync();
        
        return new PagedResult<TResult>
        {
            Data = data,
            TotalCount = totalCount,
            PageNumber = request.PageNumber,
            PageSize = request.PageSize
        };
    }
}
```

---

## 🏭 Интеграция с Unit of Work

### Интерфейс Unit of Work

```csharp
public interface IUnitOfWork : IDisposable
{
    IGenericRepository<T> Repository<T>() where T : class;
    Task<int> SaveChangesAsync();
    Task BeginTransactionAsync();
    Task CommitTransactionAsync();
    Task RollbackTransactionAsync();
}
```

### Реализация Unit of Work

```csharp
public class UnitOfWork : IUnitOfWork
{
    private readonly DbContext _context;
    private readonly Dictionary<Type, object> _repositories;
    private IDbContextTransaction? _transaction;

    public UnitOfWork(DbContext context)
    {
        _context = context ?? throw new ArgumentNullException(nameof(context));
        _repositories = new Dictionary<Type, object>();
    }

    public IGenericRepository<T> Repository<T>() where T : class
    {
        var type = typeof(T);
        
        if (!_repositories.ContainsKey(type))
        {
            _repositories[type] = new GenericRepositoryExtended<T>(_context);
        }
        
        return (IGenericRepository<T>)_repositories[type];
    }

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
        _context?.Dispose();
    }
}
```

---

## ⚙️ Регистрация в DI Container

### ASP.NET Core Startup

```csharp
// Program.cs или Startup.cs
public void ConfigureServices(IServiceCollection services)
{
    // Регистрация DbContext
    services.AddDbContext<ApplicationDbContext>(options =>
        options.UseSqlServer(connectionString));
    
    // Вариант 1: Прямая регистрация
    services.AddScoped(typeof(IGenericRepository<>), typeof(GenericRepositoryExtended<>));
    services.AddScoped<IUnitOfWork, UnitOfWork>();
    
    // Вариант 2: Фабричный метод для конкретных репозиториев
    services.AddScoped<IGenericRepository<User>>(provider =>
        new GenericRepositoryExtended<User>(provider.GetService<ApplicationDbContext>()));
}
```

---

## 💡 Примеры использования

### В сервисном слое

```csharp
public class UserService
{
    private readonly IUnitOfWork _unitOfWork;
    
    public UserService(IUnitOfWork unitOfWork)
    {
        _unitOfWork = unitOfWork;
    }
    
    public async Task<PagedResult<UserDto>> GetUsersAsync(
        int page, int size, string? search = null)
    {
        var request = new PagedRequest { PageNumber = page, PageSize = size };
        
        Expression<Func<User, bool>>? filter = null;
        if (!string.IsNullOrEmpty(search))
        {
            filter = u => u.Name.Contains(search) || u.Email.Contains(search);
        }
        
        var pagedUsers = await _unitOfWork.Repository<User>()
            .GetPagedAsync(
                request,
                u => new UserDto 
                { 
                    Id = u.Id, 
                    Name = u.Name, 
                    Email = u.Email 
                },
                filter,
                users => users.OrderBy(u => u.Name));
                
        return pagedUsers;
    }
    
    public async Task<User> CreateUserAsync(CreateUserRequest request)
    {
        await _unitOfWork.BeginTransactionAsync();
        
        try
        {
            var user = new User
            {
                Name = request.Name,
                Email = request.Email,
                CreatedAt = DateTime.UtcNow
            };
            
            await _unitOfWork.Repository<User>().AddAsync(user);
            await _unitOfWork.SaveChangesAsync();
            
            // Дополнительная логика (логи, события и т.д.)
            
            await _unitOfWork.CommitTransactionAsync();
            return user;
        }
        catch
        {
            await _unitOfWork.RollbackTransactionAsync();
            throw;
        }
    }
}
```

### В контроллере

```csharp
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    private readonly UserService _userService;
    
    public UsersController(UserService userService)
    {
        _userService = userService;
    }
    
    [HttpGet]
    public async Task<ActionResult<PagedResult<UserDto>>> GetUsers(
        [FromQuery] int page = 1,
        [FromQuery] int size = 10,
        [FromQuery] string? search = null)
    {
        var result = await _userService.GetUsersAsync(page, size, search);
        return Ok(result);
    }
}
```

---

## ⚡ Лучшие практики

> [!tip] 🎯 Рекомендации по реализации

### 1. Асинхронность

```csharp
// ✅ Хорошо - всегда async для операций с БД
public async Task<T?> GetByIdAsync(int id)
{
    return await _dbSet.FindAsync(id);
}

// ❌ Плохо - синхронные операции блокируют поток
public T GetById(int id)
{
    return _dbSet.Find(id);
}
```

### 2. Обработка связанных сущностей

```csharp
// ✅ Хорошо - контроль над загрузкой связанных данных
public async Task<User?> GetUserWithOrdersAsync(int userId)
{
    return await _unitOfWork.Repository<User>()
        .GetByIdAsync(userId, u => u.Orders);
}

// ⚠️ Внимание к N+1 проблеме
public async Task<IEnumerable<User>> GetUsersWithOrdersAsync()
{
    return await _unitOfWork.Repository<User>()
        .FindAsync(
            filter: null,
            orderBy: null,
            includes: u => u.Orders, u => u.Profile
        );
}
```

### 3. Валидация и обработка ошибок

```csharp
public async Task AddAsync(T entity)
{
    if (entity == null)
        throw new ArgumentNullException(nameof(entity));
        
    try
    {
        await _dbSet.AddAsync(entity);
    }
    catch (DbUpdateException ex)
    {
        // Логирование и обработка ошибок БД
        throw new RepositoryException("Error adding entity", ex);
    }
}
```

---

## 🎓 Выводы

> [!success] ✅ Плюсы Generic Repository
> 
> - **Повторное использование кода** - один репозиторий для всех сущностей
> - **Консистентность** - единый интерфейс для всех операций
> - **Тестируемость** - легко мокается для unit-тестов
> - **Абстракция** - скрывает детали реализации доступа к данным

> [!warning] ⚠️ Минусы и ограничения
> 
> - **Потеря специфичности** - сложно оптимизировать под конкретные сущности
> - **Overcomplexity** - может быть избыточным для простых сценариев
> - **Ограничения типобезопасности** - generic constraints могут быть недостаточными
> - **Performance** - дополнительный слой абстракции

---

## 🔗 Связанные заметки

- [[MOC - GoF - Repository|Repository]] - базовые концепции
- [[Unit of Work Pattern|#concept/unit-of-work]] - управление транзакциями
- [[Specification Pattern|#design-pattern/specification]] - инкапсуляция бизнес-правил
- [[Entity Framework Core|#tech/ef-core]] - ORM интеграция
- [[Repository Pattern - Альтернативы|Современные альтернативы]] - что вместо Repository

---

> [!note] 📝 Заметка Generic Repository - мощный инструмент, но используйте его осознанно. В большинстве случаев EF Core предоставляет достаточную функциональность без дополнительных абстракций.
