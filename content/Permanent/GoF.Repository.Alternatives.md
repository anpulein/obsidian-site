---
aliases:
  - GoF.Repository.Alternatives
  - Альтернативы
  - Alternatives
tags:
  - type/permanent
  - status/active
  - area/architecture
  - area/development
  - concept/repository
  - concept/clean-architecture
  - concept/cqrs
  - concept/med
  - concept/mediator
  - tech/csharp
  - tech/asp-net
  - tech/ef-core
  - design-pattern/reposiroty
title: GoF.Repository.Alternatives
linter-yaml-title-alias: GoF.Repository.Alternatives
date created: Tuesday, September 9th 2025, 5:48:51 am
date modified: Tuesday, September 9th 2025, 5:51:32 am
---

## 🏷️ Tags

#type/permanent #status/active #area/architecture #area/development #concept/repository #concept/clean-architecture #concept/cqrs #concept/mediator #tech/csharp #tech/asp-net #tech/ef-core #design-pattern/reposiroty 

---

# GoF.Repository.Alternatives

> [!abstract] 📋 Что изучаем
> 
> - [ ] Почему Repository Pattern критикуют в .NET
> - [ ] DbContext как естественный Repository
> - [ ] Mediator Pattern для разделения логики
> - [ ] CQRS подход к доступу к данным
> - [ ] Query Objects для сложных запросов
> - [ ] Minimal API + EF Core подход
> - [ ] Сравнение альтернатив и рекомендации

---

## 🎯 Проблема Repository Pattern в .NET

> [!warning] 🚨 Основные критические замечания
> 
> **Избыточная абстракция** — EF Core DbContext уже является реализацией Repository и Unit of Work
> 
> **Потеря функциональности** — теряем IQueryable, Include(), сложные запросы
> 
> **Дублирование кода** — множество похожих интерфейсов и реализаций
> 
> **Ложная гибкость** — редко меняем ORM в реальных проектах

---

## 🧭 Оглавление

1. [DbContext как Repository](https://claude.ai/chat/191aea21-7269-48e7-b9c0-3ff5f5cde345#-dbcontext-%D0%BA%D0%B0%D0%BA-repository)
2. [Mediator Pattern подход](https://claude.ai/chat/191aea21-7269-48e7-b9c0-3ff5f5cde345#-mediator-pattern-%D0%BF%D0%BE%D0%B4%D1%85%D0%BE%D0%B4)
3. [CQRS архитектура](https://claude.ai/chat/191aea21-7269-48e7-b9c0-3ff5f5cde345#-cqrs-%D0%B0%D1%80%D1%85%D0%B8%D1%82%D0%B5%D0%BA%D1%82%D1%83%D1%80%D0%B0)
4. [Query Objects](https://claude.ai/chat/191aea21-7269-48e7-b9c0-3ff5f5cde345#-query-objects)
5. [Service Layer подход](https://claude.ai/chat/191aea21-7269-48e7-b9c0-3ff5f5cde345#-service-layer-%D0%BF%D0%BE%D0%B4%D1%85%D0%BE%D0%B4)
6. [Minimal API стиль](https://claude.ai/chat/191aea21-7269-48e7-b9c0-3ff5f5cde345#-minimal-api-%D1%81%D1%82%D0%B8%D0%BB%D1%8C)
7. [Сравнение альтернатив](https://claude.ai/chat/191aea21-7269-48e7-b9c0-3ff5f5cde345#-%D1%81%D1%80%D0%B0%D0%B2%D0%BD%D0%B5%D0%BD%D0%B8%D0%B5-%D0%B0%D0%BB%D1%8C%D1%82%D0%B5%D1%80%D0%BD%D0%B0%D1%82%D0%B8%D0%B2)

---

## 📁 DbContext как Repository

> [!tip] 💡 Ключевая идея EF Core DbContext уже реализует Repository и Unit of Work паттерны. Зачем оборачивать готовое решение?

### Прямое использование DbContext

```csharp
// В сервисе или контроллере
public class UserService
{
    private readonly AppDbContext _context;
    
    public UserService(AppDbContext context)
    {
        _context = context;
    }
    
    public async Task<User?> GetUserWithOrdersAsync(int userId)
    {
        return await _context.Users
            .Include(u => u.Orders)
            .ThenInclude(o => o.Items)
            .FirstOrDefaultAsync(u => u.Id == userId);
    }
    
    public async Task<List<User>> GetActiveUsersAsync()
    {
        return await _context.Users
            .Where(u => u.IsActive)
            .OrderBy(u => u.Name)
            .ToListAsync();
    }
    
    public async Task CreateUserAsync(User user)
    {
        _context.Users.Add(user);
        await _context.SaveChangesAsync();
    }
}
```

### Преимущества подхода

> [!success] ✅ Плюсы
> 
> - Полный доступ к IQueryable
> - Нет лишних слоев абстракции
> - Простота и читаемость
> - Естественная работа с EF Core features

> [!danger] ❌ Минусы
> 
> - Зависимость от конкретной ORM
> - Сложнее тестирование (но решаемо с In-Memory DB)
> - Бизнес-логика смешивается с запросами

---

## 🎭 Mediator Pattern подход

> [!info] 📚 Концепция Используем [[Mediator Pattern]] для разделения команд и запросов. Популярная библиотека — **MediatR**.

### Структура с MediatR

```csharp
// Query (запрос)
public record GetUserByIdQuery(int UserId) : IRequest<User?>;

// Handler (обработчик)
public class GetUserByIdHandler : IRequestHandler<GetUserByIdQuery, User?>
{
    private readonly AppDbContext _context;
    
    public GetUserByIdHandler(AppDbContext context)
    {
        _context = context;
    }
    
    public async Task<User?> Handle(GetUserByIdQuery request, CancellationToken cancellationToken)
    {
        return await _context.Users
            .Include(u => u.Profile)
            .FirstOrDefaultAsync(u => u.Id == request.UserId, cancellationToken);
    }
}

// В контроллере
[ApiController]
public class UsersController : ControllerBase
{
    private readonly IMediator _mediator;
    
    public UsersController(IMediator mediator)
    {
        _mediator = mediator;
    }
    
    [HttpGet("{id}")]
    public async Task<ActionResult<User>> GetUser(int id)
    {
        var user = await _mediator.Send(new GetUserByIdQuery(id));
        return user is null ? NotFound() : Ok(user);
    }
}
```

### Преимущества Mediator

> [!success] ✅ Плюсы
> 
> - Четкое разделение команд и запросов
> - Легкое тестирование отдельных handlers
> - Декупление контроллеров от бизнес-логики
> - Возможность добавления cross-cutting concerns (логирование, валидация)

---

## ⚡ CQRS архитектура

> [!abstract] 🏗️ Command Query Responsibility Segregation Разделяем модели и логику для команд (изменения) и запросов (чтения).

### Структура CQRS

```csharp
// === COMMANDS (Команды) ===
public record CreateUserCommand(string Name, string Email) : IRequest<int>;

public class CreateUserCommandHandler : IRequestHandler<CreateUserCommand, int>
{
    private readonly AppDbContext _context;
    
    public async Task<int> Handle(CreateUserCommand request, CancellationToken cancellationToken)
    {
        var user = new User 
        { 
            Name = request.Name, 
            Email = request.Email,
            CreatedAt = DateTime.UtcNow
        };
        
        _context.Users.Add(user);
        await _context.SaveChangesAsync(cancellationToken);
        
        return user.Id;
    }
}

// === QUERIES (Запросы) ===
public record GetUsersListQuery(int Page, int Size) : IRequest<PagedResult<UserDto>>;

public class GetUsersListQueryHandler : IRequestHandler<GetUsersListQuery, PagedResult<UserDto>>
{
    private readonly AppDbContext _context;
    
    public async Task<PagedResult<UserDto>> Handle(GetUsersListQuery request, CancellationToken cancellationToken)
    {
        var query = _context.Users
            .Where(u => u.IsActive)
            .Select(u => new UserDto 
            { 
                Id = u.Id, 
                Name = u.Name, 
                Email = u.Email 
            });
            
        var total = await query.CountAsync(cancellationToken);
        var items = await query
            .Skip(request.Page * request.Size)
            .Take(request.Size)
            .ToListAsync(cancellationToken);
            
        return new PagedResult<UserDto>(items, total, request.Page, request.Size);
    }
}
```

### Продвинутый CQRS с разными моделями

```csharp
// Модель для записи (Command side)
public class User
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
    public DateTime CreatedAt { get; set; }
    public bool IsActive { get; set; }
}

// Модель для чтения (Query side) - может быть из другой БД
public class UserReadModel
{
    public int Id { get; set; }
    public string DisplayName { get; set; } = string.Empty;
    public string ContactInfo { get; set; } = string.Empty;
    public string Status { get; set; } = string.Empty;
    public int OrdersCount { get; set; }
    public decimal TotalSpent { get; set; }
}
```

---

## 🔍 Query Objects

> [!tip] 💡 Паттерн для сложных запросов Инкапсулируем сложную логику запросов в отдельные классы.

### Реализация Query Objects

```csharp
// Базовый интерфейс
public interface IQuery<T>
{
    IQueryable<T> Apply(IQueryable<T> query);
}

// Конкретный запрос
public class ActiveUsersWithOrdersQuery : IQuery<User>
{
    private readonly DateTime _fromDate;
    
    public ActiveUsersWithOrdersQuery(DateTime fromDate)
    {
        _fromDate = fromDate;
    }
    
    public IQueryable<User> Apply(IQueryable<User> query)
    {
        return query
            .Where(u => u.IsActive)
            .Where(u => u.Orders.Any(o => o.CreatedAt >= _fromDate))
            .Include(u => u.Orders)
            .OrderByDescending(u => u.Orders.Sum(o => o.Total));
    }
}

// Использование
public class UserService
{
    private readonly AppDbContext _context;
    
    public async Task<List<User>> GetActiveUsersWithOrdersAsync(DateTime fromDate)
    {
        var query = new ActiveUsersWithOrdersQuery(fromDate);
        
        return await query
            .Apply(_context.Users)
            .ToListAsync();
    }
}
```

### Композиция Query Objects

```csharp
// Расширение для удобства
public static class QueryableExtensions
{
    public static IQueryable<T> ApplyQuery<T>(this IQueryable<T> source, IQuery<T> query)
    {
        return query.Apply(source);
    }
}

// Использование с цепочкой
public async Task<List<User>> GetFilteredUsersAsync(
    bool onlyActive, 
    DateTime? fromDate, 
    string? nameFilter)
{
    var query = _context.Users.AsQueryable();
    
    if (onlyActive)
        query = query.ApplyQuery(new ActiveUsersQuery());
        
    if (fromDate.HasValue)
        query = query.ApplyQuery(new UsersFromDateQuery(fromDate.Value));
        
    if (!string.IsNullOrEmpty(nameFilter))
        query = query.ApplyQuery(new UsersByNameQuery(nameFilter));
    
    return await query.ToListAsync();
}
```

---

## 🔧 Service Layer подход

> [!info] 🏗️ Традиционная архитектура Выносим всю логику работы с данными в сервисный слой.

### Domain Services

```csharp
public interface IUserDomainService
{
    Task<User?> GetUserByIdAsync(int id);
    Task<User> CreateUserAsync(CreateUserRequest request);
    Task<bool> UpdateUserAsync(int id, UpdateUserRequest request);
    Task<PagedResult<UserDto>> SearchUsersAsync(UserSearchCriteria criteria);
}

public class UserDomainService : IUserDomainService
{
    private readonly AppDbContext _context;
    private readonly IMapper _mapper;
    
    public UserDomainService(AppDbContext context, IMapper mapper)
    {
        _context = context;
        _mapper = mapper;
    }
    
    public async Task<User?> GetUserByIdAsync(int id)
    {
        return await _context.Users
            .Include(u => u.Profile)
            .Include(u => u.Orders.Where(o => o.Status == OrderStatus.Active))
            .FirstOrDefaultAsync(u => u.Id == id);
    }
    
    public async Task<User> CreateUserAsync(CreateUserRequest request)
    {
        // Бизнес-логика валидации
        if (await _context.Users.AnyAsync(u => u.Email == request.Email))
            throw new BusinessException("User with this email already exists");
        
        var user = _mapper.Map<User>(request);
        user.CreatedAt = DateTime.UtcNow;
        user.IsActive = true;
        
        _context.Users.Add(user);
        await _context.SaveChangesAsync();
        
        return user;
    }
}
```

---

## 🚀 Minimal API стиль

> [!abstract] ⚡ Современный подход Упрощаем архитектуру для простых сценариев с Minimal API и прямым использованием EF Core.

### Minimal API с EF Core

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

var app = builder.Build();

// Простые endpoints
app.MapGet("/users", async (AppDbContext db) =>
    await db.Users.Where(u => u.IsActive).ToListAsync());

app.MapGet("/users/{id:int}", async (int id, AppDbContext db) =>
    await db.Users.FindAsync(id) is User user ? Results.Ok(user) : Results.NotFound());

app.MapPost("/users", async (CreateUserRequest request, AppDbContext db) =>
{
    var user = new User 
    { 
        Name = request.Name, 
        Email = request.Email,
        CreatedAt = DateTime.UtcNow,
        IsActive = true
    };
    
    db.Users.Add(user);
    await db.SaveChangesAsync();
    
    return Results.Created($"/users/{user.Id}", user);
});

app.Run();
```

### Minimal API с выносом логики

```csharp
// Статический класс для группировки endpoints
public static class UserEndpoints
{
    public static void MapUserEndpoints(this WebApplication app)
    {
        var users = app.MapGroup("/users").WithTags("Users");
        
        users.MapGet("/", GetAllUsers);
        users.MapGet("/{id:int}", GetUserById);
        users.MapPost("/", CreateUser);
        users.MapPut("/{id:int}", UpdateUser);
        users.MapDelete("/{id:int}", DeleteUser);
    }
    
    private static async Task<IResult> GetAllUsers(
        AppDbContext db, 
        [FromQuery] bool activeOnly = true)
    {
        var query = db.Users.AsQueryable();
        
        if (activeOnly)
            query = query.Where(u => u.IsActive);
        
        var users = await query.ToListAsync();
        return Results.Ok(users);
    }
    
    private static async Task<IResult> GetUserById(int id, AppDbContext db)
    {
        var user = await db.Users
            .Include(u => u.Orders)
            .FirstOrDefaultAsync(u => u.Id == id);
            
        return user is not null ? Results.Ok(user) : Results.NotFound();
    }
    
    // ... остальные методы
}
```

---

## 📊 Сравнение альтернатив

|Подход|Простота|Тестируемость|Производительность|Гибкость|Подходит для|
|---|---|---|---|---|---|
|**DbContext напрямую**|⭐⭐⭐⭐⭐|⭐⭐⭐|⭐⭐⭐⭐⭐|⭐⭐⭐|Простые CRUD|
|**Mediator/MediatR**|⭐⭐⭐|⭐⭐⭐⭐⭐|⭐⭐⭐⭐|⭐⭐⭐⭐|Средние/большие проекты|
|**CQRS**|⭐⭐|⭐⭐⭐⭐⭐|⭐⭐⭐⭐⭐|⭐⭐⭐⭐⭐|Сложные системы|
|**Query Objects**|⭐⭐⭐⭐|⭐⭐⭐⭐|⭐⭐⭐⭐|⭐⭐⭐⭐|Сложные запросы|
|**Service Layer**|⭐⭐⭐|⭐⭐⭐⭐|⭐⭐⭐|⭐⭐⭐|Традиционные проекты|
|**Minimal API**|⭐⭐⭐⭐⭐|⭐⭐|⭐⭐⭐⭐⭐|⭐⭐|Микросервисы, MVP|

---

## 🎯 Рекомендации по выбору

> [!success] 🏆 **Для большинства .NET приложений** **DbContext + Service Layer** — оптимальный баланс простоты и функциональности

> [!tip] 🚀 **Для новых проектов** **MediatR + CQRS** — современный подход с хорошей масштабируемостью

> [!warning] ⚡ **Для простых API** **Minimal API + DbContext** — максимальная производительность и простота

> [!info] 🏗️ **Для корпоративных систем** **CQRS + Event Sourcing** — когда нужна максимальная гибкость

---

## 🔗 Связанные паттерны

- [[MOC - GoF - Repository]] — исходный паттерн Repository
- [[MOC - ArchPat - CQRS|CQRS]] — разделение команд и запросов
- [[Mediator Pattern]] — медиатор для развязки
- [[GoF.Repository.Specification|Specification]] — гибкие спецификации запросов
- [[Clean Architecture - Application Layer]] — место бизнес-логики

---

## 📚 Дополнительные ресурсы

> [!example] 📖 Полезные ссылки
> 
> - [Microsoft Docs: EF Core DbContext](https://docs.microsoft.com/en-us/ef/core/dbcontext-configuration/)
> - [MediatR Documentation](https://github.com/jbogard/MediatR)
> - [CQRS Journey by Microsoft](https://docs.microsoft.com/en-us/previous-versions/msp-n-p/jj554200\(v=pandp.10\))
> - [Minimal APIs in .NET 6+](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis)

---

> [!quote] 💭 Заключение Repository Pattern не всегда зло, но в .NET экосистеме часто есть более простые и эффективные альтернативы. Выбирайте подход исходя из сложности проекта, требований к тестируемости и опыта команды.

