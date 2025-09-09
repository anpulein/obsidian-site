---
title: DDD.Repository
aliases:
  - DDD.Repository
  - Repository
linter-yaml-title-alias: DDD.Repository
date created: Friday, September 5th 2025, 3:34:53 pm
date modified: Monday, September 8th 2025, 2:29:57 pm
tags:
  - type/permanent
  - concept/ddd
  - ddd/aggregate
  - ddd/entity
  - ddd/value-object
  - area/architecture
  - area/development
  - design-pattern/factory
  - concept/immutability
  - concept/repository
  - tech/csharp
  - tech/ef-core
---
## 🏷️ Tags

#type/permanent #concept/ddd #concept/repository #tech/csharp #tech/ef-core 

---

# DDD.Repository

> [!info] 📋 О заметке Паттерн Repository в контексте Domain-Driven Design. Инкапсуляция логики доступа к данным и создание коллекции объектов в памяти.

---

## ✅ Чек-лист изучения

- [ ] Понимание назначения Repository в DDD
- [ ] Отличия от Data Access Layer
- [ ] Принципы проектирования Repository
- [ ] Связь с Aggregate и Entity
- [ ] Практические примеры реализации
- [ ] Типичные ошибки и как их избежать

---

## 📖 Содержание

1. [Определение и назначение](https://claude.ai/chat/b88ee992-fa4f-440a-8e6a-8707fa3f005d#-%D0%BE%D0%BF%D1%80%D0%B5%D0%B4%D0%B5%D0%BB%D0%B5%D0%BD%D0%B8%D0%B5-%D0%B8-%D0%BD%D0%B0%D0%B7%D0%BD%D0%B0%D1%87%D0%B5%D0%BD%D0%B8%D0%B5)
2. [Принципы Repository в DDD](https://claude.ai/chat/b88ee992-fa4f-440a-8e6a-8707fa3f005d#-%D0%BF%D1%80%D0%B8%D0%BD%D1%86%D0%B8%D0%BF%D1%8B-repository-%D0%B2-ddd)
3. [Связь с другими паттернами](https://claude.ai/chat/b88ee992-fa4f-440a-8e6a-8707fa3f005d#-%D1%81%D0%B2%D1%8F%D0%B7%D1%8C-%D1%81-%D0%B4%D1%80%D1%83%D0%B3%D0%B8%D0%BC%D0%B8-%D0%BF%D0%B0%D1%82%D1%82%D0%B5%D1%80%D0%BD%D0%B0%D0%BC%D0%B8)
4. [Примеры реализации](https://claude.ai/chat/b88ee992-fa4f-440a-8e6a-8707fa3f005d#-%D0%BF%D1%80%D0%B8%D0%BC%D0%B5%D1%80%D1%8B-%D1%80%D0%B5%D0%B0%D0%BB%D0%B8%D0%B7%D0%B0%D1%86%D0%B8%D0%B8)
5. [Типичные ошибки](https://claude.ai/chat/b88ee992-fa4f-440a-8e6a-8707fa3f005d#-%D1%82%D0%B8%D0%BF%D0%B8%D1%87%D0%BD%D1%8B%D0%B5-%D0%BE%D1%88%D0%B8%D0%B1%D0%BA%D0%B8)
6. [Лучшие практики](https://claude.ai/chat/b88ee992-fa4f-440a-8e6a-8707fa3f005d#-%D0%BB%D1%83%D1%87%D1%88%D0%B8%D0%B5-%D0%BF%D1%80%D0%B0%D0%BA%D1%82%D0%B8%D0%BA%D0%B8)

---

## 🎯 Определение и назначение

> [!abstract] 💡 Определение **Repository** - это паттерн, который инкапсулирует логику получения доменных объектов из хранилища данных, предоставляя единый интерфейс для работы с коллекцией объектов как будто они находятся в памяти.

### Основные цели Repository:

|Цель|Описание|
|---|---|
|**Изоляция домена**|Отделение бизнес-логики от деталей хранения данных|
|**Тестируемость**|Возможность легко подменить реализацию для тестов|
|**Единый интерфейс**|Консистентный способ работы с объектами|
|**Инкапсуляция запросов**|Сокрытие сложности запросов к БД|

---

## 🏗️ Принципы Repository в DDD

### 1. Один Repository на Aggregate Root

> [!tip] 🎯 Правило Repository создается только для корневых агрегатов (**Aggregate Root**), не для Entity внутри агрегата.

```csharp
// ✅ Правильно
public interface IOrderRepository
{
    Task<Order> GetByIdAsync(OrderId id);
    Task SaveAsync(Order order);
}

// ❌ Неправильно - OrderItem это Entity внутри Order
public interface IOrderItemRepository
{
    Task<OrderItem> GetByIdAsync(OrderItemId id);
}
```

### 2. Работа с бизнес-объектами

Repository оперирует доменными объектами, а не DTO или сущностями ORM:

```csharp
public interface IProductRepository
{
    // ✅ Возвращаем доменную модель
    Task<Product> GetByIdAsync(ProductId id);
    Task<IEnumerable<Product>> GetByCategoryAsync(CategoryId categoryId);
    
    // ✅ Принимаем доменную модель
    Task SaveAsync(Product product);
    Task DeleteAsync(Product product);
}
```

### 3. Коллекционная семантика

Repository работает как коллекция объектов в памяти:

```csharp
public interface ICustomerRepository
{
    // Методы как у коллекции
    Task AddAsync(Customer customer);
    Task RemoveAsync(Customer customer);
    Task<Customer> FindByIdAsync(CustomerId id);
    Task<IEnumerable<Customer>> FindByEmailAsync(string email);
    
    // Спецификации для сложных запросов
    Task<IEnumerable<Customer>> FindAsync(ISpecification<Customer> spec);
}
```

---

## 🔗 Связь с другими паттернами

### Repository + Aggregate

```csharp
public class Order // Aggregate Root
{
    public OrderId Id { get; private set; }
    private List<OrderItem> _items = new();
    
    // Методы работы с агрегатом
    public void AddItem(Product product, int quantity)
    {
        // Бизнес-логика внутри агрегата
        var item = new OrderItem(product, quantity);
        _items.Add(item);
    }
}

public interface IOrderRepository
{
    // Repository работает с целым агрегатом
    Task<Order> GetByIdAsync(OrderId id);
    Task SaveAsync(Order order); // Сохраняет весь агрегат
}
```

### Repository + Unit of Work

```csharp
public class OrderService
{
    private readonly IOrderRepository _orderRepository;
    private readonly IUnitOfWork _unitOfWork;
    
    public async Task ProcessOrderAsync(OrderId orderId)
    {
        var order = await _orderRepository.GetByIdAsync(orderId);
        order.Process();
        
        await _orderRepository.SaveAsync(order);
        await _unitOfWork.CommitAsync(); // Фиксация изменений
    }
}
```

---

## 💻 Примеры реализации

### Базовая реализация с EF Core

```csharp
public class OrderRepository : IOrderRepository
{
    private readonly ApplicationDbContext _context;
    
    public OrderRepository(ApplicationDbContext context)
    {
        _context = context;
    }
    
    public async Task<Order> GetByIdAsync(OrderId id)
    {
        var orderEntity = await _context.Orders
            .Include(o => o.Items)
            .ThenInclude(i => i.Product)
            .FirstOrDefaultAsync(o => o.Id == id.Value);
            
        return orderEntity?.ToDomainModel();
    }
    
    public async Task SaveAsync(Order order)
    {
        var existing = await _context.Orders
            .FirstOrDefaultAsync(o => o.Id == order.Id.Value);
            
        if (existing == null)
        {
            var entity = OrderEntity.FromDomain(order);
            _context.Orders.Add(entity);
        }
        else
        {
            existing.UpdateFromDomain(order);
        }
    }
}
```

### Repository с Specification Pattern

```csharp
public interface ISpecification<T>
{
    Expression<Func<T, bool>> Criteria { get; }
    List<Expression<Func<T, object>>> Includes { get; }
}

public class CustomerRepository : ICustomerRepository
{
    public async Task<IEnumerable<Customer>> FindAsync(ISpecification<Customer> spec)
    {
        var query = _context.Customers.AsQueryable();
        
        query = query.Where(spec.Criteria);
        
        query = spec.Includes.Aggregate(query, 
            (current, include) => current.Include(include));
            
        var entities = await query.ToListAsync();
        return entities.Select(e => e.ToDomainModel());
    }
}

// Использование
var activeCustomersSpec = new ActiveCustomersSpecification();
var customers = await _customerRepository.FindAsync(activeCustomersSpec);
```

---

## ⚠️ Типичные ошибки

### 1. Слишком много методов в Repository

```csharp
// ❌ Неправильно - слишком специфичные методы
public interface ICustomerRepository
{
    Task<Customer> GetByEmailAsync(string email);
    Task<Customer> GetByPhoneAsync(string phone);
    Task<IEnumerable<Customer>> GetActiveCustomersAsync();
    Task<IEnumerable<Customer>> GetCustomersByAgeRangeAsync(int min, int max);
    Task<IEnumerable<Customer>> GetVipCustomersAsync();
    // ... еще 20 методов
}

// ✅ Правильно - используем Specification
public interface ICustomerRepository
{
    Task<Customer> GetByIdAsync(CustomerId id);
    Task<IEnumerable<Customer>> FindAsync(ISpecification<Customer> spec);
    Task AddAsync(Customer customer);
    Task SaveAsync(Customer customer);
}
```

### 2. Нарушение инкапсуляции агрегата

```csharp
// ❌ Неправильно - Repository для Entity внутри агрегата
public interface IOrderItemRepository
{
    Task<OrderItem> GetByIdAsync(OrderItemId id);
    Task SaveAsync(OrderItem item); // Нарушает границы агрегата
}

// ✅ Правильно - только для Aggregate Root
public interface IOrderRepository
{
    Task<Order> GetByIdAsync(OrderId id); // Order содержит OrderItems
}
```

### 3. Возврат IQueryable

```csharp
// ❌ Неправильно - нарушает инкапсуляцию
public interface IProductRepository
{
    IQueryable<Product> GetAll(); // Логика запросов утекает в сервисы
}

// ✅ Правильно - инкапсулированные методы
public interface IProductRepository
{
    Task<IEnumerable<Product>> GetByCategoryAsync(CategoryId categoryId);
    Task<IEnumerable<Product>> FindAsync(ISpecification<Product> spec);
}
```

---

## 🎯 Лучшие практики

### 1. Используйте асинхронные методы

```csharp
public interface IRepository<TEntity, TId>
    where TEntity : IAggregateRoot<TId>
{
    Task<TEntity> GetByIdAsync(TId id);
    Task<IEnumerable<TEntity>> GetAllAsync();
    Task AddAsync(TEntity entity);
    Task SaveAsync(TEntity entity);
    Task RemoveAsync(TEntity entity);
}
```

### 2. Обрабатывайте отсутствие данных явно

```csharp
public async Task<Order> GetByIdAsync(OrderId id)
{
    var entity = await _context.Orders.FirstOrDefaultAsync(o => o.Id == id.Value);
    
    if (entity == null)
        throw new OrderNotFoundException(id);
        
    return entity.ToDomainModel();
}
```

### 3. Используйте маппинг между доменом и persistence

```csharp
// Доменная модель остается чистой
public class Order 
{
    public OrderId Id { get; private set; }
    // ... доменная логика
}

// Entity для ORM
public class OrderEntity
{
    public int Id { get; set; }
    
    public Order ToDomainModel()
    {
        // Маппинг в доменную модель
    }
    
    public static OrderEntity FromDomain(Order order)
    {
        // Маппинг из доменной модели
    }
}
```

---

## 🔗 Связанные заметки

- [[DDD.Aggregate|DDD - Aggregate]]
- [[DDD.Entity|DDD - Entity]]
- [[Unit of Work Pattern]]
- [[Specification Pattern]]
- [[MOC - DDD - Bounded Context|Bounded Context]]

---

## 📚 Дополнительные ресурсы

> [!example] 📖 Рекомендуемая литература
> 
> - "Domain-Driven Design" - Eric Evans
> - "Implementing Domain-Driven Design" - Vaughn Vernon
> - "Architecture Patterns with Python" - Harry Percival, Bob Gregory
