---
title: MS.Fundamentals
aliases:
  - MS.Fundamentals
  - Fundamentals
linter-yaml-title-alias: MS.Fundamentals
date created: Monday, September 8th 2025, 12:04:56 pm
date modified: Tuesday, September 9th 2025, 4:57:07 am
tags:
  - type/permanent
  - concept/microservice
  - concept/ddd
  - concept/clean-architecture
  - concept/database-per-service
  - concept/api-first-design
  - concept/bounded-context
  - area/architecture
  - tech/csharp
  - tech/asp-net
  - tech/ef-core
  - design-pattern/mediator
---
## 🏷️ Tags

#type/permanent #concept/microservice #concept/ddd #concept/clean-architecture #concept/database-per-service #concept/api-first-design #concept/bounded-context #area/architecture #tech/csharp #tech/asp-net #tech/ef-core #design-pattern/mediator 

---
# MS.Fundamentals

> [!info] 📋 О заметке Фундаментальные принципы и концепции построения микросервисной архитектуры в экосистеме .NET

← Назад к [[MOC - Microservices|Microservices]]

---

## ✅ Что будет раскрыто

- [ ] Определение и характеристики микросервисов в .NET
- [ ] Принципы проектирования и границы сервисов
- [ ] Структура типичного .NET микросервиса
- [ ] Сравнение с монолитом и модульным монолитом
- [ ] Инструменты и технологии экосистемы .NET

---

## 📖 Определение микросервисов

### Что НЕ является микросервисом

> [!failure] ❌ Анти-паттерны
> 
> - **Nano-services** - слишком мелкая гранулярность
> - **Distributed Monolith** - тесно связанные сервисы
> - **Shared Database** - общая база данных
> - **Synchronous Chain** - цепочка синхронных вызовов

### Правильное понимание

> [!success] ✅ Микросервис в .NET это:
> 
> - **Независимо развертываемый** компонент
> - **Владеет своими данными** (Database per Service)
> - **Реализует бизнес-capability** (не технический слой)
> - **Слабо связан** с другими сервисами
> - **Высоко связан** внутри себя

---

## 🏗️ Структура .NET микросервиса

### Рекомендуемая архитектура

```
OrderService/
├── src/
│   ├── OrderService.API/           # Presentation Layer
│   │   ├── Controllers/
│   │   ├── Middleware/
│   │   └── Program.cs
│   ├── OrderService.Application/   # Application Layer
│   │   ├── Commands/
│   │   ├── Queries/
│   │   ├── Handlers/
│   │   └── Services/
│   ├── OrderService.Domain/        # Domain Layer
│   │   ├── Entities/
│   │   ├── ValueObjects/
│   │   ├── Events/
│   │   └── Interfaces/
│   └── OrderService.Infrastructure/ # Infrastructure Layer
│       ├── Data/
│       ├── Messaging/
│       └── ExternalServices/
├── tests/
└── docker/
```

### Clean Architecture + DDD

> [!abstract] Слои архитектуры
> 
> **Domain** (Центр)
> 
> - Entities, Value Objects
> - Domain Services
> - Business Rules
> 
> **Application** (Сценарии использования)
> 
> - Command/Query Handlers
> - Application Services
> - DTOs, Mapping
> 
> **Infrastructure** (Внешние зависимости)
> 
> - Database Access
> - Message Brokers
> - External APIs
> 
> **Presentation** (API)
> 
> - Controllers
> - Middleware
> - API Contracts

---

## 🔍 Принципы проектирования

### 1. Single Responsibility Principle

```csharp
// ✅ Good: Один сервис - одна бизнес область
public class OrderService
{
    // Только логика заказов
    public Task<Order> CreateOrderAsync(CreateOrderCommand command);
    public Task<Order> UpdateOrderAsync(UpdateOrderCommand command);
    public Task CancelOrderAsync(CancelOrderCommand command);
}

// ❌ Bad: Смешанная ответственность
public class OrderUserService
{
    // Заказы И пользователи - слишком много ответственностей
}
```

### 2. Database Per Service

> [!warning] Критически важно Каждый микросервис должен иметь свою собственную базу данных. Это обеспечивает:
> 
> - Независимость развертывания
> - Технологическую свободу
> - Изолированные сбои

```yaml
# docker-compose.yml пример
services:
  order-service:
    image: orderservice:latest
    environment:
      - ConnectionString=Server=order-db;Database=Orders
  
  order-db:
    image: postgres:13
    
  catalog-service:
    image: catalogservice:latest
    environment:
      - ConnectionString=Server=catalog-db;Database=Catalog
      
  catalog-db:
    image: mongodb:latest
```

### 3. API First Design

```csharp
// Сначала определяем контракт
[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    [HttpPost]
    [ProducesResponseType(typeof(OrderDto), 201)]
    [ProducesResponseType(400)]
    public async Task<ActionResult<OrderDto>> CreateOrder(
        [FromBody] CreateOrderRequest request)
    {
        // Implementation
    }
}

// Контракт как отдельный пакет
public class OrderDto
{
    public Guid Id { get; set; }
    public string CustomerId { get; set; }
    public OrderStatus Status { get; set; }
    public List<OrderItemDto> Items { get; set; }
}
```

---

## 📊 Сравнение архитектур

|Характеристика|Монолит|Модульный монолит|Микросервисы|
|---|---|---|---|
|**Развертывание**|Единое|Единое|Независимое|
|**База данных**|Общая|Общая|Отдельная|
|**Технологии**|Единые|Единые|Различные|
|**Команда**|1 большая|1 большая|Много малых|
|**Сложность**|Низкая|Средняя|Высокая|
|**Производительность**|Высокая|Высокая|Средняя|
|**Отказоустойчивость**|Низкая|Низкая|Высокая|

---

## 🛠️ Технологический стек .NET

### Основные компоненты

> [!note] Core технологии
> 
> **Web Framework**
> 
> - ASP.NET Core 8+ (Minimal APIs, Controllers)
> - gRPC для service-to-service
> 
> **Data Access**
> 
> - Entity Framework Core
> - Dapper (для высокой производительности)
> - MongoDB.Driver, Redis
> 
> **Messaging**
> 
> - Azure Service Bus
> - RabbitMQ
> - Apache Kafka
> 
> **Monitoring**
> 
> - Application Insights
> - OpenTelemetry
> - Serilog

### Пример конфигурации сервиса

```csharp
// Program.cs - современный подход .NET 8
var builder = WebApplication.CreateBuilder(args);

// Конфигурация сервисов
builder.Services.AddControllers();
builder.Services.AddDbContext<OrderContext>(options =>
    options.UseNpgsql(builder.Configuration.GetConnectionString("DefaultConnection")));

// MediatR для CQRS
builder.Services.AddMediatR(cfg => 
    cfg.RegisterServicesFromAssembly(typeof(CreateOrderHandler).Assembly));

// Swagger для API документации
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Проверки здоровья
builder.Services.AddHealthChecks()
    .AddDbContext<OrderContext>();

var app = builder.Build();

// Middleware pipeline
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHealthChecks("/health");
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

---

## 🎯 Определение границ сервисов

### Domain-Driven Design подход

> [!tip] 💡 Как найти границы
> 
> 1. **Event Storming** - моделирование процессов через события
> 2. **Bounded Context** - определение контекстных границ
> 3. **Business Capabilities** - что делает каждая область бизнеса
> 4. **Data Ownership** - кто отвечает за какие данные

### Практический пример: E-commerce

```mermaid
graph TB
    subgraph "Catalog Context"
        A[Product Service]
    end
    
    subgraph "Order Context"  
        B[Order Service]
        C[Shopping Cart Service]
    end
    
    subgraph "Customer Context"
        D[Customer Service]
        E[Authentication Service]
    end
    
    subgraph "Inventory Context"
        F[Inventory Service]
    end
    
    subgraph "Payment Context"
        G[Payment Service]
    end
    
    B --> A : "Get Product Info"
    B --> D : "Validate Customer"  
    B --> F : "Reserve Items"
    B --> G : "Process Payment"
```

---

## 📋 Характеристики качественного микросервиса

### Принципы самостоятельности

> [!success] ✅ Autonomous Service
> 
> - **Собственная база данных** - никакого sharing
> - **Независимое развертывание** - CI/CD pipeline
> - **Fault isolation** - падение одного не ломает других
> - **Technology diversity** - выбираем лучший инструмент

### Размер и сложность

> [!question] "Насколько микро должен быть микросервис?"
> 
> **Правило двух пицц Amazon** - команда, которая может поужинать двумя пиццами
> 
> **Практические метрики:**
> 
> - 1-2 недели на полную переписывание
> - 5-9 человек в команде максимум
> - 100-1000 строк кода (очень приблизительно)
> - 1-3 бизнес функции

---

## 🚧 Типичные ошибки новичков

### 1. Распределенный монолит

```csharp
// ❌ Плохо: Сервисы слишком связаны
public class OrderService
{
    public async Task CreateOrder(CreateOrderRequest request)
    {
        // Синхронные вызовы = каскадные сбои
        var customer = await _customerService.GetCustomerAsync(request.CustomerId);
        var product = await _catalogService.GetProductAsync(request.ProductId);
        var inventory = await _inventoryService.ReserveAsync(request.ProductId);
        var payment = await _paymentService.ChargeAsync(request.PaymentInfo);
        
        // Если любой сервис недоступен - весь процесс ломается
    }
}
```

### 2. Shared Database

```csharp
// ❌ Плохо: Общая база данных
public class OrderService
{
    public DbContext SharedContext { get; set; } // Общий контекст!
}

public class CustomerService  
{
    public DbContext SharedContext { get; set; } // Тот же контекст!
}
```

### 3. Nano-services

```csharp
// ❌ Плохо: Слишком мелкая гранулярность
public class CustomerNameService { } // Только имя
public class CustomerEmailService { } // Только email  
public class CustomerAddressService { } // Только адрес

// ✅ Хорошо: Разумная гранулярность
public class CustomerService 
{
    // Вся логика управления клиентами
}
```

---

## 🎨 Варианты реализации в .NET

### Minimal APIs (.NET 6+)

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// Простая и лаконичная реализация
app.MapGet("/orders/{id:guid}", async (Guid id, IOrderService service) =>
{
    var order = await service.GetByIdAsync(id);
    return order is not null ? Results.Ok(order) : Results.NotFound();
});

app.MapPost("/orders", async (CreateOrderRequest request, IOrderService service) =>
{
    var result = await service.CreateAsync(request);
    return Results.Created($"/orders/{result.Id}", result);
});

app.Run();
```

### Controller-based API

```csharp
[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    private readonly IMediator _mediator;

    public OrdersController(IMediator mediator) => _mediator = mediator;

    [HttpPost]
    public async Task<ActionResult<OrderDto>> CreateOrder(CreateOrderCommand command)
    {
        var result = await _mediator.Send(command);
        return CreatedAtAction(nameof(GetOrder), new { id = result.Id }, result);
    }

    [HttpGet("{id:guid}")]
    public async Task<ActionResult<OrderDto>> GetOrder(Guid id)
    {
        var query = new GetOrderQuery(id);
        var result = await _mediator.Send(query);
        return result is not null ? Ok(result) : NotFound();
    }
}
```

### gRPC Services

```csharp
public class OrderGrpcService : OrderService.OrderServiceBase
{
    private readonly IMediator _mediator;

    public OrderGrpcService(IMediator mediator) => _mediator = mediator;

    public override async Task<CreateOrderResponse> CreateOrder(
        CreateOrderRequest request, ServerCallContext context)
    {
        var command = new CreateOrderCommand
        {
            CustomerId = request.CustomerId,
            Items = request.Items.Select(i => new OrderItemCommand 
            { 
                ProductId = i.ProductId, 
                Quantity = i.Quantity 
            }).ToList()
        };

        var result = await _mediator.Send(command);
        
        return new CreateOrderResponse
        {
            OrderId = result.Id.ToString(),
            Status = result.Status.ToString()
        };
    }
}
```

---

## 🚀 С чего начать

### Пошаговый план для новичков

> [!example] 📅 Roadmap (2-3 месяца)
> 
> **Неделя 1-2: Основы**
> 
> - [ ] ASP.NET Core Web API
> - [ ] Entity Framework Core
> - [ ] Docker контейнеризация
> - [ ] Создать простой монолит
> 
> **Неделя 3-4: Разделение**
> 
> - [ ] Выделить 2 сервиса из монолита
> - [ ] HTTP коммуникация между ними
> - [ ] Добавить API Gateway (Ocelot)
> 
> **Неделя 5-6: Database per Service**
> 
> - [ ] Отдельные БД для каждого сервиса
> - [ ] Async messaging (RabbitMQ)
> - [ ] Saga Pattern для транзакций
> 
> **Неделя 7-8: Production Ready**
> 
> - [ ] Logging & Monitoring
> - [ ] Health checks
> - [ ] Circuit breaker pattern
> - [ ] Kubernetes deployment

### Рекомендуемый учебный проект

```csharp
// Простой e-commerce: 3 сервиса
// 1. Catalog Service - управление товарами
// 2. Order Service - управление заказами  
// 3. User Service - управление пользователями

// Начинаем с HTTP, переходим к messaging
```

---

## 🔗 Связанные темы

- [[.NET Microservices - Architecture Patterns]] - архитектурные паттерны
- [[.NET Microservices - Communication]] - межсервисная коммуникация
- [[MOC - Clean Architcture|Clean Architecture]] - принципы чистой архитектуры
- [[MOC - DDD (Domain-Driven Design)|DDD]] - доменное моделирование
- [[MOC - ArchPat - CQRS|CQRS]] - разделение команд и запросов
- [[Arch.Event-Driven|Event-Driven Architecture]] - событийно-ориентированная архитектура

---

## 📚 Рекомендуемые ресурсы

### Документация Microsoft

- [.NET Microservices: Architecture for Containerized .NET Applications](https://docs.microsoft.com/en-us/dotnet/architecture/microservices/)
- [eShopOnContainers Reference Application](https://github.com/dotnet-architecture/eShopOnContainers)

### Книги

- **"Microservices Patterns"** - Chris Richardson
- **"Building Microservices"** - Sam Newman
- **"Microservices in .NET"** - Christian Horsdal

### Примеры кода

- [eShopOnContainers](https://github.com/dotnet-architecture/eShopOnContainers) - референсная реализация Microsoft
- [Clean Architecture Solution Template](https://github.com/jasontaylordev/CleanArchitecture) - шаблон Jason Taylor

---

> [!success] 🎯 Ключевые выводы
> 
> 1. **Начинайте с модульного монолита** - не прыгайте сразу в микросервисы
> 2. **Database per service** - фундаментальное требование
> 3. **Бизнес-capability**, не технические слои - основа для разделения
> 4. **Async communication** предпочтительнее sync где возможно
> 5. **.NET 8+ отлично подходит** для микросервисной архитектуры
> 6. **DevOps зрелость критична** - без неё микросервисы превратятся в кошмар