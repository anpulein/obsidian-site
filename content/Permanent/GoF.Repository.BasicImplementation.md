---
aliases:
  - GoF.Repository.BasicImplementation
  - Basic Implementation
  - Базовая реализация
tags:
  - type/permanent
  - status/active
  - area/development
  - area/architecture
  - tech/csharp
  - tech/ef-core
  - concept/repository
  - design-pattern/reposiroty
  - design-pattern/dependency-injection
  - concept/clean-architecture
  - concept/ddd
  - concept/solid
  - impl/ef-core
title: GoF.Repository.BasicImplementation
linter-yaml-title-alias: GoF.Repository.BasicImplementation
date created: Tuesday, September 9th 2025, 5:15:17 am
date modified: Tuesday, September 9th 2025, 5:22:45 am
---

## 🏷️ Tags

#type/permanent #status/active #area/development #area/architecture #tech/csharp #tech/ef-core #concept/repository #design-pattern/reposiroty #design-pattern/dependency-injection #concept/clean-architecture #concept/ddd #concept/solid #impl/ef-core 

---

# GoF.Repository.BasicImplementation

> [!info]- 📋 Краткий чек-лист изучения
> 
> - ✅ Понять назначение и преимущества паттерна Repository
> - ✅ Изучить базовую структуру интерфейсов
> - ✅ Рассмотреть реализацию Generic Repository
> - ✅ Понять интеграцию с Entity Framework Core
> - ✅ Изучить регистрацию в DI Container
> - ✅ Рассмотреть практические примеры использования
> - ✅ Понять ограничения и альтернативы

---

## 🎯 Содержание

1. [[#Что такое Repository Pattern]]
2. [[#Базовая структура]]
3. [[#Generic Repository]]
4. [[#Specific Repository]]
5. [[#Интеграция с EF Core]]
6. [[#Регистрация в DI]]
7. [[#Практические примеры]]
8. [[#Альтернативы и ограничения]]

---

## 📚 Что такое Repository Pattern

> [!abstract] Определение **Repository Pattern** — это паттерн проектирования, который инкапсулирует логику доступа к данным и обеспечивает более объектно-ориентированное представление слоя персистентности.

### Основные цели

|Цель|Описание|
|---|---|
|**Абстракция**|Скрывает детали реализации доступа к данным|
|**Тестируемость**|Позволяет легко подменять реализацию для тестов|
|**Централизация**|Собирает логику запросов в одном месте|
|**Независимость**|Отделяет бизнес-логику от технологии доступа к данным|

---

## 🏗️ Базовая структура

### IRepository<`T`> - Generic интерфейс

```csharp
public interface IRepository<T> where T : class
{
    // Чтение
    Task<T?> GetByIdAsync(int id, CancellationToken cancellationToken = default);
    Task<IEnumerable<T>> GetAllAsync(CancellationToken cancellationToken = default);
    Task<IEnumerable<T>> FindAsync(Expression<Func<T, bool>> predicate, CancellationToken cancellationToken = default);
    
    // Запись
    Task<T> AddAsync(T entity, CancellationToken cancellationToken = default);
    Task<IEnumerable<T>> AddRangeAsync(IEnumerable<T> entities, CancellationToken cancellationToken = default);
    
    // Обновление
    Task UpdateAsync(T entity, CancellationToken cancellationToken = default);
    Task UpdateRangeAsync(IEnumerable<T> entities, CancellationToken cancellationToken = default);
    
    // Удаление
    Task DeleteAsync(T entity, CancellationToken cancellationToken = default);
    Task DeleteRangeAsync(IEnumerable<T> entities, CancellationToken cancellationToken = default);
    Task DeleteByIdAsync(int id, CancellationToken cancellationToken = default);
    
    // Утилиты
    Task<int> CountAsync(CancellationToken cancellationToken = default);
    Task<int> CountAsync(Expression<Func<T, bool>> predicate, CancellationToken cancellationToken = default);
    Task<bool> ExistsAsync(int id, CancellationToken cancellationToken = default);
}
```

---

## 🔧 Generic Repository

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
    
    public async Task<T?> GetByIdAsync(int id, CancellationToken cancellationToken = default)
    {
        return await _dbSet.FindAsync(new object[] { id }, cancellationToken);
    }
    
    public async Task<IEnumerable<T>> GetAllAsync(CancellationToken cancellationToken = default)
    {
        return await _dbSet.ToListAsync(cancellationToken);
    }
    
    public async Task<IEnumerable<T>> FindAsync(Expression<Func<T, bool>> predicate, CancellationToken cancellationToken = default)
    {
        return await _dbSet.Where(predicate).ToListAsync(cancellationToken);
    }
    
    public async Task<T> AddAsync(T entity, CancellationToken cancellationToken = default)
    {
        await _dbSet.AddAsync(entity, cancellationToken);
        return entity;
    }
    
    public async Task<IEnumerable<T>> AddRangeAsync(IEnumerable<T> entities, CancellationToken cancellationToken = default)
    {
        await _dbSet.AddRangeAsync(entities, cancellationToken);
        return entities;
    }
    
    public Task UpdateAsync(T entity, CancellationToken cancellationToken = default)
    {
        _dbSet.Update(entity);
        return Task.CompletedTask;
    }
    
    public Task UpdateRangeAsync(IEnumerable<T> entities, CancellationToken cancellationToken = default)
    {
        _dbSet.UpdateRange(entities);
        return Task.CompletedTask;
    }
    
    public Task DeleteAsync(T entity, CancellationToken cancellationToken = default)
    {
        _dbSet.Remove(entity);
        return Task.CompletedTask;
    }
    
    public Task DeleteRangeAsync(IEnumerable<T> entities, CancellationToken cancellationToken = default)
    {
        _dbSet.RemoveRange(entities);
        return Task.CompletedTask;
    }
    
    public async Task DeleteByIdAsync(int id, CancellationToken cancellationToken = default)
    {
        var entity = await GetByIdAsync(id, cancellationToken);
        if (entity != null)
        {
            await DeleteAsync(entity, cancellationToken);
        }
    }
    
    public async Task<int> CountAsync(CancellationToken cancellationToken = default)
    {
        return await _dbSet.CountAsync(cancellationToken);
    }
    
    public async Task<int> CountAsync(Expression<Func<T, bool>> predicate, CancellationToken cancellationToken = default)
    {
        return await _dbSet.CountAsync(predicate, cancellationToken);
    }
    
    public async Task<bool> ExistsAsync(int id, CancellationToken cancellationToken = default)
    {
        return await _dbSet.FindAsync(new object[] { id }, cancellationToken) != null;
    }
}
```

---

## 📝 Specific Repository

### Интерфейс для конкретной сущности

```csharp
public interface IUserRepository : IRepository<User>
{
    Task<User?> GetByEmailAsync(string email, CancellationToken cancellationToken = default);
    Task<IEnumerable<User>> GetActiveUsersAsync(CancellationToken cancellationToken = default);
    Task<IEnumerable<User>> GetUsersByRoleAsync(string role, CancellationToken cancellationToken = default);
    Task<bool> EmailExistsAsync(string email, CancellationToken cancellationToken = default);
}
```

### Реализация специфичного репозитория

```csharp
public class UserRepository : Repository<User>, IUserRepository
{
    public UserRepository(ApplicationDbContext context) : base(context) { }
    
    public async Task<User?> GetByEmailAsync(string email, CancellationToken cancellationToken = default)
    {
        return await _dbSet.FirstOrDefaultAsync(u => u.Email == email, cancellationToken);
    }
    
    public async Task<IEnumerable<User>> GetActiveUsersAsync(CancellationToken cancellationToken = default)
    {
        return await _dbSet
            .Where(u => u.IsActive)
            .ToListAsync(cancellationToken);
    }
    
    public async Task<IEnumerable<User>> GetUsersByRoleAsync(string role, CancellationToken cancellationToken = default)
    {
        return await _dbSet
            .Where(u => u.Role == role)
            .ToListAsync(cancellationToken);
    }
    
    public async Task<bool> EmailExistsAsync(string email, CancellationToken cancellationToken = default)
    {
        return await _dbSet.AnyAsync(u => u.Email == email, cancellationToken);
    }
}
```

---

## 🔗 Интеграция с EF Core

### Модель сущности

```csharp
public class User
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
    public string Role { get; set; } = "User";
    public bool IsActive { get; set; } = true;
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
    public DateTime? UpdatedAt { get; set; }
}
```

### DbContext

```csharp
public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options) : base(options) { }
    
    public DbSet<User> Users { get; set; }
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<User>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.HasIndex(e => e.Email).IsUnique();
            entity.Property(e => e.Name).IsRequired().HasMaxLength(100);
            entity.Property(e => e.Email).IsRequired().HasMaxLength(255);
        });
    }
}
```

---

## 🏭 Регистрация в DI

### Простая регистрация

```csharp
// Program.cs
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

// Регистрация репозиториев
builder.Services.AddScoped(typeof(IRepository<>), typeof(Repository<>));
builder.Services.AddScoped<IUserRepository, UserRepository>();
```

### Расширенная регистрация с использованием Extension Method

```csharp
public static class ServiceCollectionExtensions
{
    public static IServiceCollection AddRepositories(this IServiceCollection services)
    {
        // Generic Repository
        services.AddScoped(typeof(IRepository<>), typeof(Repository<>));
        
        // Specific Repositories
        services.AddScoped<IUserRepository, UserRepository>();
        // services.AddScoped<IProductRepository, ProductRepository>();
        // services.AddScoped<IOrderRepository, OrderRepository>();
        
        return services;
    }
}

// Program.cs
builder.Services.AddRepositories();
```

---

## 💡 Практические примеры

### Использование в контроллере

```csharp
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    private readonly IUserRepository _userRepository;
    private readonly ApplicationDbContext _context;
    
    public UsersController(IUserRepository userRepository, ApplicationDbContext context)
    {
        _userRepository = userRepository;
        _context = context;
    }
    
    [HttpGet]
    public async Task<ActionResult<IEnumerable<User>>> GetUsers()
    {
        var users = await _userRepository.GetAllAsync();
        return Ok(users);
    }
    
    [HttpGet("{id}")]
    public async Task<ActionResult<User>> GetUser(int id)
    {
        var user = await _userRepository.GetByIdAsync(id);
        if (user == null)
            return NotFound();
        
        return Ok(user);
    }
    
    [HttpPost]
    public async Task<ActionResult<User>> CreateUser(User user)
    {
        // Проверяем уникальность email
        if (await _userRepository.EmailExistsAsync(user.Email))
        {
            return Conflict("Email already exists");
        }
        
        var createdUser = await _userRepository.AddAsync(user);
        await _context.SaveChangesAsync(); // Сохраняем изменения
        
        return CreatedAtAction(nameof(GetUser), new { id = createdUser.Id }, createdUser);
    }
    
    [HttpPut("{id}")]
    public async Task<IActionResult> UpdateUser(int id, User user)
    {
        if (id != user.Id)
            return BadRequest();
        
        var existingUser = await _userRepository.GetByIdAsync(id);
        if (existingUser == null)
            return NotFound();
        
        user.UpdatedAt = DateTime.UtcNow;
        await _userRepository.UpdateAsync(user);
        await _context.SaveChangesAsync();
        
        return NoContent();
    }
    
    [HttpDelete("{id}")]
    public async Task<IActionResult> DeleteUser(int id)
    {
        var user = await _userRepository.GetByIdAsync(id);
        if (user == null)
            return NotFound();
        
        await _userRepository.DeleteAsync(user);
        await _context.SaveChangesAsync();
        
        return NoContent();
    }
}
```

### Использование в сервисе

```csharp
public interface IUserService
{
    Task<User?> CreateUserAsync(CreateUserRequest request);
    Task<User?> GetUserByEmailAsync(string email);
    Task<bool> DeactivateUserAsync(int userId);
}

public class UserService : IUserService
{
    private readonly IUserRepository _userRepository;
    private readonly ApplicationDbContext _context;
    
    public UserService(IUserRepository userRepository, ApplicationDbContext context)
    {
        _userRepository = userRepository;
        _context = context;
    }
    
    public async Task<User?> CreateUserAsync(CreateUserRequest request)
    {
        // Бизнес-логика валидации
        if (await _userRepository.EmailExistsAsync(request.Email))
            return null;
        
        var user = new User
        {
            Name = request.Name,
            Email = request.Email,
            Role = request.Role ?? "User",
            IsActive = true
        };
        
        await _userRepository.AddAsync(user);
        await _context.SaveChangesAsync();
        
        return user;
    }
    
    public async Task<User?> GetUserByEmailAsync(string email)
    {
        return await _userRepository.GetByEmailAsync(email);
    }
    
    public async Task<bool> DeactivateUserAsync(int userId)
    {
        var user = await _userRepository.GetByIdAsync(userId);
        if (user == null) return false;
        
        user.IsActive = false;
        user.UpdatedAt = DateTime.UtcNow;
        
        await _userRepository.UpdateAsync(user);
        await _context.SaveChangesAsync();
        
        return true;
    }
}
```

---

## ⚠️ Альтернативы и ограничения

### Ограничения Repository Pattern

> [!warning]- Основные проблемы
> 
> - **Дублирование функциональности** - EF Core уже является Repository
> - **Сложность тестирования** - DbContext можно мокать напрямую
> - **Потеря производительности** - дополнительный слой абстракции
> - **IQueryable vs IEnumerable** - потеря ленивых вычислений

### Современные альтернативы

|Подход|Описание|Когда использовать|
|---|---|---|
|**Direct EF Core**|Прямое использование DbContext|Простые CRUD операции|
|**MediatR Pattern**|CQRS с медиатором|Сложная бизнес-логика|
|**Specification Pattern**|Инкапсуляция запросов|Сложные критерии поиска|

### Пример без Repository

```csharp
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    private readonly ApplicationDbContext _context;
    
    public UsersController(ApplicationDbContext context)
    {
        _context = context;
    }
    
    [HttpGet]
    public async Task<ActionResult<IEnumerable<User>>> GetUsers()
    {
        return await _context.Users.ToListAsync();
    }
    
    [HttpGet("{id}")]
    public async Task<ActionResult<User>> GetUser(int id)
    {
        var user = await _context.Users.FindAsync(id);
        return user == null ? NotFound() : Ok(user);
    }
}
```

---

## 📖 Связанные темы

- [[Unit of Work Pattern]] - управление транзакциями
- [[Specification Pattern]] - инкапсуляция бизнес-правил
- [[MOC - ArchPat - CQRS|CQRS]] - разделение команд и запросов
- [[MediatR Pattern]] - медиатор для обработки запросов
- [[Entity Framework Core]] - ORM для .NET

---

> [!success] Заключение Repository Pattern в .NET полезен для создания четкого слоя абстракции над доступом к данным, особенно в сложных проектах с множественными источниками данных. Однако при использовании EF Core стоит взвешивать необходимость дополнительного слоя против его преимуществ.