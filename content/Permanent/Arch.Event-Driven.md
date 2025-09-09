---
title: Arch.Event-Driven Architecture
aliases:
  - Arch.Event-Driven Architecture
  - Event-Driven Architecture
linter-yaml-title-alias: Arch.Event-Driven Architecture
date created: Monday, September 8th 2025, 7:21:26 am
date modified: Tuesday, September 9th 2025, 5:10:42 am
tags:
  - type/permanent
  - concept/event
  - concept/event-driven
  - concept/microservice
  - concept/eventual-cinsistenct
  - concept/eventual-consistency
  - concept/observability
  - tech/csharp
  - tech/asp-net
  - area/architecture
---
## 🏷️ Tags

#type/permanent #concept/event-driven #concept/microservice #concept/eventual-consistency #concept/observability #tech/csharp #tech/asp-net #area/architecture  

---

# Arch.Event-Driven Architecture

> [!info] 📋 О документе Комплексное руководство по построению событийно-ориентированной архитектуры в .NET экосистеме

---

## 🎯 Что изучим

- [x] **Основы EDA** - ключевые концепции и принципы
- [x] **Компоненты архитектуры** - события, обработчики, брокеры
- [x] **Паттерны реализации** - Publisher/Subscriber, Event Sourcing
- [x] **Инструменты .NET** - MediatR, SignalR, массивы событий
- [x] **Практические примеры** - от простых до enterprise-решений
- [x] **Мониторинг и отладка** - трейсинг событий, метрики
- [x] **Best Practices** - ошибки и их решения

---

## 📖 Содержание

1. [[#🎭 Основы Event-Driven Architecture]]
2. [[#🏗️ Компоненты EDA]]
3. [[#🔄 Паттерны реализации]]
4. [[#🛠️ Инструменты .NET]]
5. [[#💡 Практические примеры]]
6. [[#📊 Мониторинг и диагностика]]
7. [[#✨ Best Practices]]
8. [[#🔗 Связанные концепции]]

---

## 🎭 Основы Event-Driven Architecture

> [!tip] 💡 Ключевая идея EDA базируется на **производстве**, **обнаружении** и **реагировании** на события в реальном времени

### Определение

**Event-Driven Architecture (EDA)** — архитектурный паттерн, где компоненты системы взаимодействуют через **асинхронные события**, обеспечивая слабую связанность и высокую масштабируемость.

### Основные принципы

| Принцип            | Описание                                   | Польза                     |
| ------------------ | ------------------------------------------ | -------------------------- |
| **Loose Coupling** | Компоненты не знают друг о друге напрямую  | Независимое развитие       |
| **Asynchronous**   | Обработка событий не блокирует отправителя | Высокая производительность |
| **Scalability**    | Легкое масштабирование обработчиков        | Обработка нагрузки         |
| **Resilience**     | Устойчивость к сбоям отдельных компонентов | Надежность системы         |

---

## 🏗️ Компоненты EDA

### Event (Событие)

```csharp
public abstract record DomainEvent
{
    public Guid Id { get; init; } = Guid.NewGuid();
    public DateTime OccurredAt { get; init; } = DateTime.UtcNow;
    public string Source { get; init; } = string.Empty;
}

public record OrderCreated(
    Guid OrderId,
    decimal Amount,
    string CustomerEmail
) : DomainEvent;
```

### Event Publisher (Издатель)

```csharp
public interface IEventPublisher
{
    Task PublishAsync<T>(T @event) where T : DomainEvent;
    Task PublishManyAsync<T>(IEnumerable<T> events) where T : DomainEvent;
}

public class EventPublisher : IEventPublisher
{
    private readonly IMediator _mediator;
    
    public async Task PublishAsync<T>(T @event) where T : DomainEvent
    {
        await _mediator.Publish(@event);
    }
}
```

### Event Handler (Обработчик)

```csharp
public class OrderCreatedHandler : INotificationHandler<OrderCreated>
{
    private readonly IEmailService _emailService;
    private readonly ILogger<OrderCreatedHandler> _logger;
    
    public async Task Handle(OrderCreated notification, CancellationToken cancellationToken)
    {
        _logger.LogInformation("Processing order {OrderId}", notification.OrderId);
        
        await _emailService.SendConfirmationAsync(
            notification.CustomerEmail, 
            notification.OrderId
        );
    }
}
```

---

## 🔄 Паттерны реализации

### 1. In-Process Events (MediatR)

> [!success] ✅ Преимущества
> 
> - Простота реализации
> - Транзакционность
> - Отсутствие сетевых задержек

```csharp
// Domain Entity
public class Order
{
    private readonly List<DomainEvent> _domainEvents = new();
    
    public void Create(decimal amount, string customerEmail)
    {
        // Business logic
        _domainEvents.Add(new OrderCreated(Id, amount, customerEmail));
    }
    
    public IReadOnlyCollection<DomainEvent> DomainEvents => _domainEvents.AsReadOnly();
}

// Infrastructure
public class EventDispatcher
{
    public async Task DispatchEventsAsync(Entity entity)
    {
        foreach (var domainEvent in entity.DomainEvents)
        {
            await _mediator.Publish(domainEvent);
        }
    }
}
```

### 2. Message Bus Pattern

```csharp
public class ServiceBusEventPublisher : IEventPublisher
{
    private readonly ServiceBusClient _client;
    private readonly ServiceBusSender _sender;
    
    public async Task PublishAsync<T>(T @event) where T : DomainEvent
    {
        var message = new ServiceBusMessage(JsonSerializer.Serialize(@event))
        {
            Subject = typeof(T).Name,
            MessageId = @event.Id.ToString()
        };
        
        await _sender.SendMessageAsync(message);
    }
}
```

### 3. Event Sourcing Integration

```csharp
public class EventStore
{
    public async Task AppendEventsAsync(string streamId, IEnumerable<DomainEvent> events)
    {
        foreach (var @event in events)
        {
            var eventData = new EventData(
                Guid.NewGuid(),
                @event.GetType().Name,
                Encoding.UTF8.GetBytes(JsonSerializer.Serialize(@event))
            );
            
            await _eventStoreClient.AppendToStreamAsync(streamId, StreamState.Any, new[] { eventData });
        }
    }
}
```

---

## 🛠️ Инструменты .NET

### MediatR для In-Process Events

```csharp
// Startup.cs / Program.cs
builder.Services.AddMediatR(cfg => cfg.RegisterServicesFromAssembly(typeof(Program).Assembly));

// Pipeline Behavior для логирования
public class LoggingBehavior<TRequest, TResponse> : IPipelineBehavior<TRequest, TResponse>
{
    public async Task<TResponse> Handle(TRequest request, RequestHandlerDelegate<TResponse> next, CancellationToken cancellationToken)
    {
        _logger.LogInformation("Handling {RequestName}", typeof(TRequest).Name);
        var response = await next();
        _logger.LogInformation("Handled {RequestName}", typeof(TRequest).Name);
        return response;
    }
}
```

### SignalR для Real-time Events

```csharp
public class NotificationHub : Hub
{
    public async Task JoinGroup(string groupName)
    {
        await Groups.AddToGroupAsync(Context.ConnectionId, groupName);
    }
}

public class OrderEventHandler : INotificationHandler<OrderCreated>
{
    private readonly IHubContext<NotificationHub> _hubContext;
    
    public async Task Handle(OrderCreated notification, CancellationToken cancellationToken)
    {
        await _hubContext.Clients.Group("orders")
            .SendAsync("OrderCreated", notification, cancellationToken);
    }
}
```

### Azure Service Bus Integration

```csharp
public class ServiceBusEventConsumer : BackgroundService
{
    private readonly ServiceBusProcessor _processor;
    
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        _processor.ProcessMessageAsync += ProcessMessageAsync;
        _processor.ProcessErrorAsync += ProcessErrorAsync;
        
        await _processor.StartProcessingAsync(stoppingToken);
    }
    
    private async Task ProcessMessageAsync(ProcessMessageEventArgs args)
    {
        var eventType = args.Message.Subject;
        var eventData = args.Message.Body.ToString();
        
        // Deserialize and handle event
        await HandleEventAsync(eventType, eventData);
        await args.CompleteMessageAsync(args.Message);
    }
}
```

---

## 💡 Практические примеры

### E-commerce Order Processing

```csharp
// События жизненного цикла заказа
public record OrderCreated(Guid OrderId, decimal Amount, string CustomerEmail) : DomainEvent;
public record PaymentProcessed(Guid OrderId, Guid PaymentId) : DomainEvent;
public record OrderShipped(Guid OrderId, string TrackingNumber) : DomainEvent;

// Saga для управления процессом
public class OrderProcessingSaga : 
    INotificationHandler<OrderCreated>,
    INotificationHandler<PaymentProcessed>
{
    public async Task Handle(OrderCreated notification, CancellationToken cancellationToken)
    {
        // 1. Резервируем товар
        await _inventoryService.ReserveAsync(notification.OrderId);
        
        // 2. Инициируем платеж
        await _paymentService.ProcessAsync(notification.OrderId, notification.Amount);
    }
    
    public async Task Handle(PaymentProcessed notification, CancellationToken cancellationToken)
    {
        // 3. Отправляем заказ
        await _shippingService.ShipAsync(notification.OrderId);
        
        // 4. Публикуем событие отправки
        await _eventPublisher.PublishAsync(new OrderShipped(
            notification.OrderId, 
            await _shippingService.GetTrackingNumberAsync(notification.OrderId)
        ));
    }
}
```

### CQRS с Events

```csharp
// Command Handler с событиями
public class CreateOrderHandler : IRequestHandler<CreateOrderCommand, Guid>
{
    public async Task<Guid> Handle(CreateOrderCommand request, CancellationToken cancellationToken)
    {
        var order = Order.Create(request.Amount, request.CustomerEmail);
        
        await _repository.AddAsync(order);
        
        // Dispatch domain events
        await _eventDispatcher.DispatchEventsAsync(order);
        
        return order.Id;
    }
}

// Read Model Projector
public class OrderProjectionHandler : INotificationHandler<OrderCreated>
{
    public async Task Handle(OrderCreated notification, CancellationToken cancellationToken)
    {
        var projection = new OrderProjection
        {
            Id = notification.OrderId,
            Amount = notification.Amount,
            CustomerEmail = notification.CustomerEmail,
            Status = "Created",
            CreatedAt = notification.OccurredAt
        };
        
        await _projectionRepository.AddAsync(projection);
    }
}
```

---

## 📊 Мониторинг и диагностика

### Event Tracing

```csharp
public class TracingEventPublisher : IEventPublisher
{
    private readonly IEventPublisher _innerPublisher;
    private readonly ActivitySource _activitySource;
    
    public async Task PublishAsync<T>(T @event) where T : DomainEvent
    {
        using var activity = _activitySource.StartActivity($"Event.Publish.{typeof(T).Name}");
        
        activity?.SetTag("event.id", @event.Id.ToString());
        activity?.SetTag("event.type", typeof(T).Name);
        activity?.SetTag("event.source", @event.Source);
        
        try
        {
            await _innerPublisher.PublishAsync(@event);
            activity?.SetStatus(ActivityStatusCode.Ok);
        }
        catch (Exception ex)
        {
            activity?.SetStatus(ActivityStatusCode.Error, ex.Message);
            throw;
        }
    }
}
```

### Metrics Collection

```csharp
public class MetricsEventPublisher : IEventPublisher
{
    private readonly Counter<int> _eventsPublished;
    private readonly Histogram<double> _publishDuration;
    
    public MetricsEventPublisher(IMeterFactory meterFactory)
    {
        var meter = meterFactory.Create("EventDriven.Events");
        _eventsPublished = meter.CreateCounter<int>("events_published_total");
        _publishDuration = meter.CreateHistogram<double>("event_publish_duration_ms");
    }
    
    public async Task PublishAsync<T>(T @event) where T : DomainEvent
    {
        var stopwatch = Stopwatch.StartNew();
        
        try
        {
            await _innerPublisher.PublishAsync(@event);
            _eventsPublished.Add(1, new("event_type", typeof(T).Name));
        }
        finally
        {
            _publishDuration.Record(stopwatch.ElapsedMilliseconds, new("event_type", typeof(T).Name));
        }
    }
}
```

### Health Checks

```csharp
public class EventProcessingHealthCheck : IHealthCheck
{
    private readonly IEventStore _eventStore;
    
    public async Task<HealthCheckResult> CheckHealthAsync(HealthCheckContext context, CancellationToken cancellationToken = default)
    {
        try
        {
            var lastEvents = await _eventStore.GetLastEventsAsync(10);
            var oldestEvent = lastEvents.OrderBy(e => e.OccurredAt).FirstOrDefault();
            
            if (oldestEvent != null && DateTime.UtcNow - oldestEvent.OccurredAt > TimeSpan.FromMinutes(5))
            {
                return HealthCheckResult.Degraded("Event processing is lagging");
            }
            
            return HealthCheckResult.Healthy();
        }
        catch (Exception ex)
        {
            return HealthCheckResult.Unhealthy("Event store is not available", ex);
        }
    }
}
```

---

## ✨ Best Practices

### 🎯 Проектирование событий

> [!warning] ⚠️ Важно События должны быть **неизменяемыми** и содержать всю необходимую информацию

```csharp
// ✅ Хорошо - самодостаточное событие
public record CustomerRegistered(
    Guid CustomerId,
    string Email,
    string FirstName,
    string LastName,
    DateTime RegisteredAt
) : DomainEvent;

// ❌ Плохо - требует дополнительных запросов
public record CustomerRegistered(Guid CustomerId) : DomainEvent;
```

### 🔄 Idempotency

```csharp
public class IdempotentEventHandler : INotificationHandler<OrderCreated>
{
    private readonly IDistributedCache _cache;
    
    public async Task Handle(OrderCreated notification, CancellationToken cancellationToken)
    {
        var idempotencyKey = $"order-created-{notification.OrderId}";
        
        if (await _cache.GetStringAsync(idempotencyKey) != null)
        {
            _logger.LogInformation("Event {EventId} already processed", notification.Id);
            return;
        }
        
        // Process event
        await ProcessOrderAsync(notification);
        
        // Mark as processed
        await _cache.SetStringAsync(idempotencyKey, "processed", 
            new DistributedCacheEntryOptions
            {
                AbsoluteExpirationRelativeToNow = TimeSpan.FromHours(24)
            });
    }
}
```

### 🛡️ Error Handling

```csharp
public class ResilientEventHandler : INotificationHandler<DomainEvent>
{
    public async Task Handle(DomainEvent notification, CancellationToken cancellationToken)
    {
        var retryPolicy = Policy
            .Handle<Exception>()
            .WaitAndRetryAsync(
                retryCount: 3,
                sleepDurationProvider: retryAttempt => TimeSpan.FromSeconds(Math.Pow(2, retryAttempt)),
                onRetry: (outcome, timespan, retryCount, context) =>
                {
                    _logger.LogWarning("Retry {RetryCount} for event {EventId} after {Delay}ms", 
                        retryCount, notification.Id, timespan.TotalMilliseconds);
                });
        
        await retryPolicy.ExecuteAsync(async () =>
        {
            await ProcessEventAsync(notification);
        });
    }
}
```

### 📋 Версионирование событий

```csharp
public abstract record DomainEvent
{
    public int Version { get; init; } = 1;
}

public record CustomerRegisteredV1(
    Guid CustomerId,
    string Email
) : DomainEvent;

public record CustomerRegisteredV2(
    Guid CustomerId,
    string Email,
    string FirstName,
    string LastName
) : DomainEvent { public override int Version { get; init; } = 2; }

// Handler с поддержкой версий
public class CustomerRegisteredHandler : 
    INotificationHandler<CustomerRegisteredV1>,
    INotificationHandler<CustomerRegisteredV2>
{
    public async Task Handle(CustomerRegisteredV1 notification, CancellationToken cancellationToken)
    {
        // Handle legacy version
        await ProcessLegacyRegistration(notification);
    }
    
    public async Task Handle(CustomerRegisteredV2 notification, CancellationToken cancellationToken)
    {
        // Handle current version
        await ProcessCurrentRegistration(notification);
    }
}
```

---

## 🚨 Антипаттерны и решения

|Антипаттерн|Проблема|Решение|
|---|---|---|
|**Event Chain Hell**|Длинные цепочки событий|Используйте Saga Pattern|
|**Missing Events**|Потеря событий при сбоях|Outbox Pattern + транзакции|
|**Dual Writes**|Несогласованность данных и событий|Event Sourcing или Outbox|
|**Fat Events**|Переполненные данными события|Разделите на множество событий|
|**Synchronous Processing**|Блокирующая обработка|Асинхронные обработчики|

---

## 🔗 Связанные концепции

### Внутренние связи

- [[MOC - ArchPat - CQRS]] - разделение команд и запросов
- [[ArchPat.EventSourcing]] - хранение событий как источника истины
- [[Eventual Consistency]] - согласованность через время
- [[Mediator]] - посредник для обработки событий
- [[Saga]] - управление распределенными транзакциями

### Технологии

- [[MediatR]] - in-process медиатор
- [[ASP.NET Core]] - веб-фреймворк
- [[Microservices]] - распределенная архитектура

---

> [!success] 🎯 Итоги Event-Driven Architecture в .NET обеспечивает **слабую связанность**, **масштабируемость** и **отказоустойчивость** через асинхронное взаимодействие компонентов на основе событий. Ключ к успеху — правильное проектирование событий, надежная обработка ошибок и качественный мониторинг.