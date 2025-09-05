---
title: 🎯 Value Object
aliases:
  - 🎯 Value Object
linter-yaml-title-alias: 🎯 Value Object
date created: Friday, September 5th 2025, 10:07:51 am
date modified: Friday, September 5th 2025, 10:19:12 am
tags:
  - type/permanent
  - concept/ddd
  - ddd/entity
  - area/architecture
  - area/development
  - ddd/value-object
  - design-pattern/factory
  - concept/immutability
---
## 🏷️ Tags

#type/permanent #concept/ddd #ddd/value-object  #area/architecture #area/development #design-pattern/factory  #concept/immutability 

---
# 🎯 Value Object

> [!abstract] 💡 Краткое описание Value Object (Объект-значение) — неизменяемый объект, который описывается только своими атрибутами, без концептуальной идентичности. Равенство определяется значениями всех свойств.

---

## 🎯 Чек-лист изучения

- [ ] Понимание концепции и отличий от Entity
- [ ] Принципы проектирования Value Object
- [ ] Реализация в коде (C#)
- [ ] Паттерны использования в доменной модели
- [ ] Валидация и обработка ошибок
- [ ] Интеграция с [[DDD/Entity|Entity]] и [[DDD/Aggregate|Aggregate]]

---

## 📋 Содержание

- [[#🔍 Основные концепции]]
- [[#⚖️ Value Object vs Entity]]
- [[#🏗️ Принципы проектирования]]
- [[#💻 Реализация в C#]]
- [[#🎯 Паттерны использования]]
- [[#🔗 Связанные концепции]]

---

## 🔍 Основные концепции

> [!info] 🎯 Ключевые характеристики
> 
> - **Неизменяемость** — после создания объект не может быть изменен
> - **Структурное равенство** — два объекта равны, если равны все их свойства
> - **Отсутствие идентичности** — объект описывается только своими значениями
> - **Самовалидация** — объект всегда находится в валидном состоянии

### Примеры из реального мира

|Value Object|Описание|Ключевые свойства|
|---|---|---|
|Money|Деньги|Amount, Currency|
|Email|Email адрес|Value|
|PersonName|Имя человека|FirstName, LastName|
|Address|Адрес|Street, City, PostalCode|
|DateRange|Диапазон дат|StartDate, EndDate|

---

## ⚖️ Value Object vs Entity

|Аспект|Value Object|Entity|
|---|---|---|
|**Идентичность**|Отсутствует|Есть уникальный Id|
|**Равенство**|По значениям всех свойств|По идентификатору|
|**Изменяемость**|Неизменяемый|Может изменяться|
|**Жизненный цикл**|Создается и уничтожается|Отслеживается во времени|
|**Назначение**|Описывает характеристики|Представляет сущность|

> [!example] 💰 Пример различия
> 
> - **Money(100, "USD")** — Value Object (важна сумма и валюта)
> - **BankAccount(Id: 12345)** — Entity (важна идентичность счета)

---

## 🏗️ Принципы проектирования

### 1. 🔒 Неизменяемость (Immutability)

> [!tip] ✨ Лучшая практика Все свойства должны быть `readonly` или `init-only`, а методы изменения должны возвращать новый экземпляр.

### 2. 🛡️ Самовалидация

```csharp
public class Email
{
    public string Value { get; init; }
    
    public Email(string value)
    {
        if (string.IsNullOrWhiteSpace(value))
            throw new ArgumentException("Email cannot be empty");
            
        if (!IsValidEmail(value))
            throw new ArgumentException("Invalid email format");
            
        Value = value;
    }
}
```

### 3. ⚖️ Структурное равенство

```csharp
public override bool Equals(object obj)
{
    if (obj is not Email other) return false;
    return Value == other.Value;
}

public override int GetHashCode() => Value.GetHashCode();
```

### 4. 🏭 Фабричные методы

```csharp
public static class Money
{
    public static Money Zero(string currency) => new(0, currency);
    public static Money FromCents(int cents, string currency) => new(cents / 100m, currency);
}
```

---

## 💻 Реализация в C#

### 📝 Базовая реализация

```csharp
public record Money(decimal Amount, string Currency)
{
    public Money 
    {
        get 
        {
            if (Amount < 0)
                throw new ArgumentException("Amount cannot be negative");
            if (string.IsNullOrEmpty(Currency))
                throw new ArgumentException("Currency is required");
        }
    }
    
    public Money Add(Money other)
    {
        if (Currency != other.Currency)
            throw new InvalidOperationException("Cannot add different currencies");
        
        return new Money(Amount + other.Amount, Currency);
    }
    
    public bool IsZero() => Amount == 0;
}
```

### 🎯 С использованием record

```csharp
public record PersonName(string FirstName, string LastName)
{
    public PersonName
    {
        get
        {
            if (string.IsNullOrWhiteSpace(FirstName))
                throw new ArgumentException("First name is required");
            if (string.IsNullOrWhiteSpace(LastName))
                throw new ArgumentException("Last name is required");
        }
    }
    
    public string FullName => $"{FirstName} {LastName}";
    
    public static PersonName Parse(string fullName)
    {
        var parts = fullName.Split(' ', 2);
        if (parts.Length != 2)
            throw new ArgumentException("Invalid full name format");
        
        return new PersonName(parts[0], parts[1]);
    }
}
```

### 🏭 Сложный Value Object

```csharp
public record Address
{
    public string Street { get; init; }
    public string City { get; init; }
    public string PostalCode { get; init; }
    public string Country { get; init; }
    
    public Address(string street, string city, string postalCode, string country)
    {
        Street = Guard.NotEmpty(street, nameof(street));
        City = Guard.NotEmpty(city, nameof(city));
        PostalCode = ValidatePostalCode(postalCode);
        Country = Guard.NotEmpty(country, nameof(country));
    }
    
    private static string ValidatePostalCode(string postalCode)
    {
        if (string.IsNullOrWhiteSpace(postalCode))
            throw new ArgumentException("Postal code is required");
        
        // Дополнительная валидация по стране
        return postalCode;
    }
    
    public string FormattedAddress => $"{Street}, {City}, {PostalCode}, {Country}";
}
```

---

## 🎯 Паттерны использования

### 1. 🏢 В составе Entity

```csharp
public class Customer : Entity<CustomerId>
{
    public PersonName Name { get; private set; }
    public Email Email { get; private set; }
    public Address Address { get; private set; }
    
    public void ChangeEmail(Email newEmail)
    {
        Email = newEmail;
        // Логика бизнес-правил
    }
}
```

### 2. 📊 Составные Value Objects

```csharp
public record OrderLine
{
    public ProductName Product { get; init; }
    public Quantity Quantity { get; init; }
    public Money UnitPrice { get; init; }
    
    public Money TotalPrice => UnitPrice.Multiply(Quantity.Value);
}
```

### 3. 🔄 Коллекции Value Objects

```csharp
public record ShoppingCart
{
    private readonly List<OrderLine> _lines = new();
    
    public IReadOnlyList<OrderLine> Lines => _lines.AsReadOnly();
    
    public Money TotalAmount => _lines
        .Select(line => line.TotalPrice)
        .Aggregate(Money.Zero("USD"), (sum, price) => sum.Add(price));
}
```

---

## 🚫 Распространенные ошибки

> [!warning] ⚠️ Частые ошибки
> 
> - **Изменяемые свойства** — нарушение принципа неизменяемости
> - **Отсутствие валидации** — невалидные состояния объекта
> - **Использование как Entity** — добавление идентификатора
> - **Примитивная навязчивость** — использование примитивов вместо VO

---

## 🔗 Связанные концепции

- [[DDD/Entity|Entity]] — сущности с идентичностью
- [[DDD/Aggregate|Aggregate]] — границы консистентности
- [[DDD - Domain Service|Domain Service]] — доменные операции
- [[|Immutability]] — неизменяемость объектов
- [[Factory Pattern]] — создание объектов

---

> [!quote] 💭 Принцип "Value Objects — это атомы доменной модели. Они несут смысл, защищают инварианты и делают код выразительнее." — Eric Evans