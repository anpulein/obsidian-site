---
title: DDD - Domain Specifications
aliases:
  - DDD - Domain Specifications
linter-yaml-title-alias: DDD - Domain Specifications
date created: Friday, August 22nd 2025, 6:17:27 am
date modified: Friday, August 22nd 2025, 6:22:12 am
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

> [!abstract] **Что это?** Domain Specifications (Спецификации домена) — это паттерн проектирования в DDD, который инкапсулирует бизнес-правила и условия в переиспользуемые объекты. Позволяет выразить сложную бизнес-логику декларативно и композируемо.

## 🎯 Основные концепции

### Зачем нужны спецификации?

- **Переиспользование** бизнес-правил
- **Инкапсуляция** сложной логики
- **Композиция** условий
- **Читаемость** кода
- **Тестируемость** правил

---

## 🏗️ Базовая структура

### Интерфейс спецификации

```csharp
public interface ISpecification<T>
{
    bool IsSatisfiedBy(T candidate);
    
    // Для композиции
    ISpecification<T> And(ISpecification<T> other);
    ISpecification<T> Or(ISpecification<T> other);
    ISpecification<T> Not();
}
```

### Абстрактная реализация

```csharp
public abstract class Specification<T> : ISpecification<T>
{
    public abstract bool IsSatisfiedBy(T candidate);

    public ISpecification<T> And(ISpecification<T> other)
    {
        return new AndSpecification<T>(this, other);
    }

    public ISpecification<T> Or(ISpecification<T> other)
    {
        return new OrSpecification<T>(this, other);
    }

    public ISpecification<T> Not()
    {
        return new NotSpecification<T>(this);
    }
}
```

---

## 🧩 Композитные спецификации

### And-спецификация

```csharp
public class AndSpecification<T> : Specification<T>
{
    private readonly ISpecification<T> _left;
    private readonly ISpecification<T> _right;

    public AndSpecification(ISpecification<T> left, ISpecification<T> right)
    {
        _left = left;
        _right = right;
    }

    public override bool IsSatisfiedBy(T candidate)
    {
        return _left.IsSatisfiedBy(candidate) && _right.IsSatisfiedBy(candidate);
    }
}
```

### Or-спецификация

```csharp
public class OrSpecification<T> : Specification<T>
{
    private readonly ISpecification<T> _left;
    private readonly ISpecification<T> _right;

    public OrSpecification(ISpecification<T> left, ISpecification<T> right)
    {
        _left = left;
        _right = right;
    }

    public override bool IsSatisfiedBy(T candidate)
    {
        return _left.IsSatisfiedBy(candidate) || _right.IsSatisfiedBy(candidate);
    }
}
```

### Not-спецификация

```csharp
public class NotSpecification<T> : Specification<T>
{
    private readonly ISpecification<T> _specification;

    public NotSpecification(ISpecification<T> specification)
    {
        _specification = specification;
    }

    public override bool IsSatisfiedBy(T candidate)
    {
        return !_specification.IsSatisfiedBy(candidate);
    }
}
```

---

## 📝 Практический пример: Система заказов

### Модель домена

```csharp
public class Customer
{
    public int Id { get; private set; }
    public string Name { get; private set; }
    public CustomerType Type { get; private set; }
    public DateTime RegistrationDate { get; private set; }
    public bool IsActive { get; private set; }
    public decimal TotalOrdersAmount { get; private set; }
    
    // Constructor and methods...
}

public enum CustomerType
{
    Regular,
    Premium,
    VIP
}
```

### Конкретные спецификации

> [!info] **Активный клиент** Клиент должен быть активным в системе

```csharp
public class ActiveCustomerSpecification : Specification<Customer>
{
    public override bool IsSatisfiedBy(Customer customer)
    {
        return customer.IsActive;
    }
}
```

> [!info] **Премиум клиент** Клиент должен иметь премиум статус

```csharp
public class PremiumCustomerSpecification : Specification<Customer>
{
    public override bool IsSatisfiedBy(Customer customer)
    {
        return customer.Type == CustomerType.Premium || 
               customer.Type == CustomerType.VIP;
    }
}
```

> [!info] **Лояльный клиент** Клиент зарегистрирован более года назад

```csharp
public class LoyalCustomerSpecification : Specification<Customer>
{
    public override bool IsSatisfiedBy(Customer customer)
    {
        return customer.RegistrationDate <= DateTime.Now.AddYears(-1);
    }
}
```

> [!info] **Крупный клиент** Общая сумма заказов превышает определенную границу

```csharp
public class HighValueCustomerSpecification : Specification<Customer>
{
    private readonly decimal _minimumAmount;

    public HighValueCustomerSpecification(decimal minimumAmount = 10000)
    {
        _minimumAmount = minimumAmount;
    }

    public override bool IsSatisfiedBy(Customer customer)
    {
        return customer.TotalOrdersAmount >= _minimumAmount;
    }
}
```

---

## 🎯 Использование спецификаций

### В доменных сервисах

```csharp
public class DiscountService
{
    public decimal CalculateDiscount(Customer customer)
    {
        var eligibleForDiscount = new ActiveCustomerSpecification()
            .And(new PremiumCustomerSpecification())
            .And(new LoyalCustomerSpecification());

        if (eligibleForDiscount.IsSatisfiedBy(customer))
        {
            return 0.15m; // 15% скидка
        }

        var standardDiscount = new ActiveCustomerSpecification()
            .And(new HighValueCustomerSpecification(5000));

        if (standardDiscount.IsSatisfiedBy(customer))
        {
            return 0.05m; // 5% скидка
        }

        return 0m;
    }
}
```

### В репозиториях (с Expression)

> [!tip] **Продвинутый подход** Для работы с Entity Framework можно расширить спецификации Expression-деревьями

```csharp
public interface ISpecification<T>
{
    bool IsSatisfiedBy(T candidate);
    Expression<Func<T, bool>> ToExpression();
}

public class ActiveCustomerSpecification : Specification<Customer>
{
    public override bool IsSatisfiedBy(Customer customer)
    {
        return customer.IsActive;
    }

    public override Expression<Func<Customer, bool>> ToExpression()
    {
        return customer => customer.IsActive;
    }
}
```

### Использование в репозитории

```csharp
public class CustomerRepository
{
    private readonly DbContext _context;

    public CustomerRepository(DbContext context)
    {
        _context = context;
    }

    public async Task<List<Customer>> FindBySpecificationAsync(
        ISpecification<Customer> specification)
    {
        return await _context.Customers
            .Where(specification.ToExpression())
            .ToListAsync();
    }
}
```

---

## ✅ Преимущества

|Преимущество|Описание|
|---|---|
|**Переиспользование**|Одна спецификация может использоваться в разных местах|
|**Композиция**|Легко комбинировать простые правила в сложные|
|**Читаемость**|Бизнес-правила выражены декларативно|
|**Тестируемость**|Каждое правило можно протестировать изолированно|
|**Инкапсуляция**|Сложная логика скрыта за простым интерфейсом|

---

## ⚠️ Недостатки и ограничения

> [!warning] **Осторожно!**
> 
> - Может привести к избыточной сложности для простых случаев
> - Требует дополнительных усилий для интеграции с ORM
> - Может снизить производительность при неправильном использовании

---

## 🧪 Тестирование спецификаций

```csharp
[Test]
public void ActiveCustomerSpecification_ShouldReturnTrue_WhenCustomerIsActive()
{
    // Arrange
    var customer = new Customer { IsActive = true };
    var specification = new ActiveCustomerSpecification();

    // Act
    var result = specification.IsSatisfiedBy(customer);

    // Assert
    Assert.IsTrue(result);
}

[Test]
public void CompositeSpecification_ShouldWork_WithAndOperator()
{
    // Arrange
    var customer = new Customer 
    { 
        IsActive = true, 
        Type = CustomerType.Premium 
    };
    
    var specification = new ActiveCustomerSpecification()
        .And(new PremiumCustomerSpecification());

    // Act
    var result = specification.IsSatisfiedBy(customer);

    // Assert
    Assert.IsTrue(result);
}
```

---

## 🏆 Лучшие практики

> [!success] **Рекомендации**
> 
> 1. **Именуйте спецификации** в соответствии с бизнес-терминологией
> 2. **Делайте их неизменяемыми** (immutable)
> 3. **Тестируйте каждую спецификацию** отдельно
> 4. **Используйте композицию** вместо наследования
> 5. **Кешируйте результаты** для дорогих операций

---

## 🔗 Связанные паттерны

- [[02 - Areas/Architecture/DDD/Repository Pattern|Repository Pattern]] - для фильтрации данных
- [[Domain Services|Domain Services]] - для сложной бизнес-логики
- [[Entities and Value Objects|Entities and Value Objects]] - для инкапсуляции правил
- [[Policy Pattern]] - альтернативный подход к правилам

---

## 📚 Дополнительные материалы

> [!note] **Для изучения**
> 
> - Eric Evans "Domain-Driven Design"
> - Martin Fowler "Specification Pattern"
> - Vaughn Vernon "Implementing Domain-Driven Design"