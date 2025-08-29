---
title: Query Handlers
aliases:
  - Query Handlers
linter-yaml-title-alias: Query Handlers
date created: Friday, August 22nd 2025, 7:52:48 am
date modified: Friday, August 22nd 2025, 7:55:14 am
tags:
  - type/area
  - area/architecture
  - concept/microservice
  - concept/clean-architecture
  - concept/ddd
---
## 🏷️ Tags

#type/area #area/architecture #concept/microservice #concept/clean-architecture #concept/ddd 

---

> [!info] Что такое Query Handlers? **Query Handlers** в Domain-Driven Design — это компоненты, которые отвечают за обработку запросов на чтение данных. Они инкапсулируют логику получения информации и возвращают данные в нужном формате без изменения состояния системы.

## 📋 Содержание

- [[#Основные концепции]]
- [[#Структура Query Handler]]
- [[#Практические примеры]]
- [[#Паттерны и лучшие практики]]
- [[#Интеграция с CQRS]]

---

## 🎯 Основные концепции

### Принципы Query Handlers

```mermaid
graph LR
    A[Query] --> B[Query Handler]
    B --> C[Repository/Data Access]
    C --> D[Domain Model]
    D --> E[DTO/View Model]
    E --> F[Response]
```

> [!note] Ключевые характеристики
> 
> - **Только чтение** - не изменяют состояние системы
> - **Специализация** - каждый handler отвечает за конкретный запрос
> - **Изоляция** - отделены от команд (Commands)
> - **Оптимизация** - могут использовать специализированные модели чтения

---

## 🏗️ Структура Query Handler

### 1. Базовый интерфейс

```csharp
public interface IQuery<out TResult>
{
    // Маркерный интерфейс для запросов
}

public interface IQueryHandler<in TQuery, TResult> 
    where TQuery : IQuery<TResult>
{
    Task<TResult> HandleAsync(TQuery query, CancellationToken cancellationToken = default);
}
```

### 2. Пример Query

```csharp
public class GetUserByIdQuery : IQuery<UserDto>
{
    public Guid UserId { get; }
    
    public GetUserByIdQuery(Guid userId)
    {
        UserId = userId;
    }
}

public class UserDto
{
    public Guid Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
    public DateTime CreatedAt { get; set; }
}
```

### 3. Реализация Handler

```csharp
public class GetUserByIdQueryHandler : IQueryHandler<GetUserByIdQuery, UserDto>
{
    private readonly IUserRepository _userRepository;
    private readonly IMapper _mapper;
    
    public GetUserByIdQueryHandler(IUserRepository userRepository, IMapper mapper)
    {
        _userRepository = userRepository;
        _mapper = mapper;
    }
    
    public async Task<UserDto> HandleAsync(GetUserByIdQuery query, CancellationToken cancellationToken = default)
    {
        var user = await _userRepository.GetByIdAsync(query.UserId, cancellationToken);
        
        if (user == null)
            throw new UserNotFoundException(query.UserId);
            
        return _mapper.Map<UserDto>(user);
    }
}
```

---

## 💡 Практические примеры

### Простой запрос списка

> [!example] Получение списка продуктов
> 
> ```csharp
> public class GetProductsQuery : IQuery<IEnumerable<ProductDto>>
> {
>     public int PageSize { get; }
>     public int PageNumber { get; }
>     public string Category { get; }
>     
>     public GetProductsQuery(int pageSize, int pageNumber, string category = null)
>     {
>         PageSize = pageSize;
>         PageNumber = pageNumber;
>         Category = category;
>     }
> }
> 
> public class GetProductsQueryHandler : IQueryHandler<GetProductsQuery, IEnumerable<ProductDto>>
> {
>     private readonly IProductRepository _productRepository;
>     
>     public GetProductsQueryHandler(IProductRepository productRepository)
>     {
>         _productRepository = productRepository;
>     }
>     
>     public async Task<IEnumerable<ProductDto>> HandleAsync(GetProductsQuery query, CancellationToken cancellationToken = default)
>     {
>         var products = await _productRepository.GetPagedAsync(
>             query.PageNumber, 
>             query.PageSize, 
>             query.Category, 
>             cancellationToken);
>             
>         return products.Select(p => new ProductDto
>         {
>             Id = p.Id,
>             Name = p.Name,
>             Price = p.Price,
>             Category = p.Category.Name
>         });
>     }
> }
> ```

### Сложный запрос с агрегацией

> [!example] Отчет по продажам
> 
> ```csharp
> public class GetSalesReportQuery : IQuery<SalesReportDto>
> {
>     public DateTime From { get; }
>     public DateTime To { get; }
>     public Guid? ProductId { get; }
>     
>     public GetSalesReportQuery(DateTime from, DateTime to, Guid? productId = null)
>     {
>         From = from;
>         To = to;
>         ProductId = productId;
>     }
> }
> 
> public class SalesReportDto
> {
>     public decimal TotalRevenue { get; set; }
>     public int TotalOrders { get; set; }
>     public decimal AverageOrderValue { get; set; }
>     public List<ProductSalesDto> ProductSales { get; set; }
> }
> 
> public class GetSalesReportQueryHandler : IQueryHandler<GetSalesReportQuery, SalesReportDto>
> {
>     private readonly IOrderRepository _orderRepository;
>     
>     public GetSalesReportQueryHandler(IOrderRepository orderRepository)
>     {
>         _orderRepository = orderRepository;
>     }
>     
>     public async Task<SalesReportDto> HandleAsync(GetSalesReportQuery query, CancellationToken cancellationToken = default)
>     {
>         var orders = await _orderRepository.GetOrdersInPeriodAsync(
>             query.From, 
>             query.To, 
>             query.ProductId, 
>             cancellationToken);
>         
>         return new SalesReportDto
>         {
>             TotalRevenue = orders.Sum(o => o.TotalAmount),
>             TotalOrders = orders.Count(),
>             AverageOrderValue = orders.Average(o => o.TotalAmount),
>             ProductSales = orders
>                 .SelectMany(o => o.Items)
>                 .GroupBy(i => i.ProductId)
>                 .Select(g => new ProductSalesDto
>                 {
>                     ProductId = g.Key,
>                     TotalQuantity = g.Sum(i => i.Quantity),
>                     TotalRevenue = g.Sum(i => i.Price * i.Quantity)
>                 })
>                 .ToList()
>         };
>     }
> }
> ```

---

## 🔧 Паттерны и лучшие практики

### Query Dispatcher

> [!tip] Централизованная обработка запросов
> 
> ```csharp
> public interface IQueryDispatcher
> {
>     Task<TResult> DispatchAsync<TResult>(IQuery<TResult> query, CancellationToken cancellationToken = default);
> }
> 
> public class QueryDispatcher : IQueryDispatcher
> {
>     private readonly IServiceProvider _serviceProvider;
>     
>     public QueryDispatcher(IServiceProvider serviceProvider)
>     {
>         _serviceProvider = serviceProvider;
>     }
>     
>     public async Task<TResult> DispatchAsync<TResult>(IQuery<TResult> query, CancellationToken cancellationToken = default)
>     {
>         var handlerType = typeof(IQueryHandler<,>).MakeGenericType(query.GetType(), typeof(TResult));
>         var handler = _serviceProvider.GetRequiredService(handlerType);
>         
>         var method = handlerType.GetMethod(nameof(IQueryHandler<IQuery<TResult>, TResult>.HandleAsync));
>         var result = await (Task<TResult>)method.Invoke(handler, new object[] { query, cancellationToken });
>         
>         return result;
>     }
> }
> ```

### Кеширование результатов

> [!warning] Декоратор для кеширования
> 
> ```csharp
> public class CachedQueryHandler<TQuery, TResult> : IQueryHandler<TQuery, TResult>
>     where TQuery : IQuery<TResult>
> {
>     private readonly IQueryHandler<TQuery, TResult> _handler;
>     private readonly IMemoryCache _cache;
>     private readonly TimeSpan _cacheTime;
>     
>     public CachedQueryHandler(IQueryHandler<TQuery, TResult> handler, IMemoryCache cache, TimeSpan cacheTime)
>     {
>         _handler = handler;
>         _cache = cache;
>         _cacheTime = cacheTime;
>     }
>     
>     public async Task<TResult> HandleAsync(TQuery query, CancellationToken cancellationToken = default)
>     {
>         var cacheKey = GenerateCacheKey(query);
>         
>         if (_cache.TryGetValue(cacheKey, out TResult cachedResult))
>             return cachedResult;
>         
>         var result = await _handler.HandleAsync(query, cancellationToken);
>         
>         _cache.Set(cacheKey, result, _cacheTime);
>         
>         return result;
>     }
>     
>     private string GenerateCacheKey(TQuery query)
>     {
>         return $"{typeof(TQuery).Name}:{JsonSerializer.Serialize(query)}";
>     }
> }
> ```

---

## 🔄 Интеграция с CQRS

### Регистрация в DI контейнере

```csharp
public static class ServiceCollectionExtensions
{
    public static IServiceCollection AddQueryHandlers(this IServiceCollection services, Assembly assembly)
    {
        // Регистрация всех Query Handlers
        services.Scan(scan => scan
            .FromAssemblies(assembly)
            .AddClasses(classes => classes
                .AssignableTo(typeof(IQueryHandler<,>)))
            .AsImplementedInterfaces()
            .WithTransientLifetime());
        
        // Регистрация диспетчера
        services.AddScoped<IQueryDispatcher, QueryDispatcher>();
        
        return services;
    }
}
```

### Использование в контроллере

```csharp
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    private readonly IQueryDispatcher _queryDispatcher;
    
    public UsersController(IQueryDispatcher queryDispatcher)
    {
        _queryDispatcher = queryDispatcher;
    }
    
    [HttpGet("{id:guid}")]
    public async Task<ActionResult<UserDto>> GetUser(Guid id)
    {
        var query = new GetUserByIdQuery(id);
        var user = await _queryDispatcher.DispatchAsync(query);
        
        return Ok(user);
    }
    
    [HttpGet]
    public async Task<ActionResult<IEnumerable<ProductDto>>> GetProducts(
        [FromQuery] int pageSize = 10,
        [FromQuery] int pageNumber = 1,
        [FromQuery] string category = null)
    {
        var query = new GetProductsQuery(pageSize, pageNumber, category);
        var products = await _queryDispatcher.DispatchAsync(query);
        
        return Ok(products);
    }
}
```

---

## ✅ Преимущества и недостатки

> [!success] Преимущества
> 
> - **Разделение ответственности** - четкое разделение логики чтения и записи
> - **Тестируемость** - легко писать unit-тесты для отдельных handlers
> - **Масштабируемость** - можно оптимизировать каждый handler независимо
> - **Читаемость** - код становится более понятным и структурированным

> [!failure] Недостатки
> 
> - **Сложность** - увеличение количества классов и интерфейсов
> - **Overhead** - дополнительные слои абстракции
> - **Дублирование** - возможное дублирование логики между handlers

---

## 📚 Связанные темы

- [[CQRS Pattern]]
- [[Domain Services]]
- [[Repository Pattern]]
- [[DTO и View Models]]
- [[Dependency Injection]]

---

> [!quote] Помни Query Handlers должны быть **быстрыми**, **простыми** и **специализированными**. Один handler — одна задача чтения данных.