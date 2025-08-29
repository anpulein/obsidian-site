---
title: Entities and Value Objects
aliases:
  - Entities and Value Objects
linter-yaml-title-alias: Entities and Value Objects
date created: Friday, August 22nd 2025, 5:51:21 am
date modified: Friday, August 22nd 2025, 7:52:15 am
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

> [!info] Domain-Driven Design (DDD) Entities и Value Objects - это два ключевых строительных блока в Domain-Driven Design, которые помогают правильно моделировать бизнес-логику.

## 🎯 Ключевые различия

|Характеристика|Entity|Value Object|
|---|---|---|
|**Идентичность**|✅ Имеет уникальный идентификатор|❌ Определяется только значениями|
|**Изменяемость**|✅ Может изменяться во времени|❌ Неизменяемый (immutable)|
|**Сравнение**|По ID|По всем свойствам|
|**Жизненный цикл**|Длительный|Создается и уничтожается|

---

## 🏛️ Entity (Сущность)

> [!tip] Определение **Entity** - объект, который имеет уникальную идентичность и может изменяться во времени, сохраняя при этом свою идентичность.

### 📋 Характеристики Entity:

- [ ] Имеет уникальный идентификатор
- [ ] Может изменять свои атрибуты
- [ ] Сравнивается по ID, а не по значениям
- [ ] Имеет жизненный цикл

### 💻 Пример Entity в .NET

```csharp
public class User : IEntity
{
    public Guid Id { get; private set; }
    public string Email { get; private set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public DateTime CreatedAt { get; private set; }
    
    public User(string email, string firstName, string lastName)
    {
        Id = Guid.NewGuid();
        Email = email;
        FirstName = firstName;
        LastName = lastName;
        CreatedAt = DateTime.UtcNow;
    }
    
    public void ChangeEmail(string newEmail)
    {
        // Бизнес-логика валидации
        if (string.IsNullOrWhiteSpace(newEmail))
            throw new ArgumentException("Email cannot be empty");
            
        Email = newEmail;
    }
    
    // Сравнение по ID
    public override bool Equals(object obj)
    {
        if (obj is User other)
            return Id == other.Id;
        return false;
    }
    
    public override int GetHashCode() => Id.GetHashCode();
}
```

> [!example] Примеры Entity
> 
> - 👤 **User** (пользователь)
> - 📦 **Order** (заказ)
> - 🏪 **Product** (товар)
> - 🏦 **BankAccount** (банковский счет)

---

## 💎 Value Object (Объект-значение)

> [!tip] Определение **Value Object** - неизменяемый объект, который определяется исключительно своими значениями и не имеет уникальной идентичности.

### 📋 Характеристики Value Object:

- [ ] Не имеет идентификатора
- [ ] Неизменяемый (immutable)
- [ ] Сравнивается по всем свойствам
- [ ] Легко создается и уничтожается

### 💻 Пример Value Object в .NET

```csharp
public record Address
{
    public string Street { get; init; }
    public string City { get; init; }
    public string PostalCode { get; init; }
    public string Country { get; init; }
    
    public Address(string street, string city, string postalCode, string country)
    {
        Street = street ?? throw new ArgumentNullException(nameof(street));
        City = city ?? throw new ArgumentNullException(nameof(city));
        PostalCode = postalCode ?? throw new ArgumentNullException(nameof(postalCode));
        Country = country ?? throw new ArgumentNullException(nameof(country));
    }
    
    // Валидация в конструкторе
    public static Address Create(string street, string city, string postalCode, string country)
    {
        if (string.IsNullOrWhiteSpace(street))
            throw new ArgumentException("Street cannot be empty");
        if (string.IsNullOrWhiteSpace(city))
            throw new ArgumentException("City cannot be empty");
            
        return new Address(street, city, postalCode, country);
    }
}

// Альтернативный пример с классом
public class Money : IEquatable<Money>
{
    public decimal Amount { get; }
    public string Currency { get; }
    
    public Money(decimal amount, string currency)
    {
        if (amount < 0)
            throw new ArgumentException("Amount cannot be negative");
        if (string.IsNullOrWhiteSpace(currency))
            throw new ArgumentException("Currency cannot be empty");
            
        Amount = amount;
        Currency = currency;
    }
    
    public Money Add(Money other)
    {
        if (Currency != other.Currency)
            throw new InvalidOperationException("Cannot add different currencies");
            
        return new Money(Amount + other.Amount, Currency);
    }
    
    public bool Equals(Money other)
    {
        if (other is null) return false;
        return Amount == other.Amount && Currency == other.Currency;
    }
    
    public override bool Equals(object obj) => Equals(obj as Money);
    public override int GetHashCode() => HashCode.Combine(Amount, Currency);
}
```

> [!example] Примеры Value Objects
> 
> - 📍 **Address** (адрес)
> - 💰 **Money** (деньги)
> - 📧 **Email** (email адрес)
> - 📅 **DateRange** (диапазон дат)
> - 🎨 **Color** (цвет)

---

## 🔍 Практическое применение

### 🏗️ Комбинированный пример

```csharp
// Entity
public class Customer
{
    public Guid Id { get; private set; }
    public Email Email { get; private set; }  // Value Object
    public Address Address { get; private set; }  // Value Object
    public List<Order> Orders { get; private set; } = new();
    
    public Customer(Email email, Address address)
    {
        Id = Guid.NewGuid();
        Email = email;
        Address = address;
    }
    
    public void ChangeAddress(Address newAddress)
    {
        Address = newAddress ?? throw new ArgumentNullException(nameof(newAddress));
    }
}

// Value Object для Email
public record Email
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
    
    private static bool IsValidEmail(string email)
    {
        return System.Text.RegularExpressions.Regex.IsMatch(
            email, @"^[^@\s]+@[^@\s]+\.[^@\s]+$");
    }
    
    public static implicit operator string(Email email) => email.Value;
    public static implicit operator Email(string value) => new(value);
}
```

---

## ❓ Как выбрать между Entity и Value Object?

> [!question] Задайте себе вопросы:

### 🤔 Это Entity, если:

- ✅ Объект имеет уникальную идентичность?
- ✅ Важно отслеживать изменения во времени?
- ✅ Объект имеет жизненный цикл?
- ✅ Два объекта с одинаковыми данными могут быть разными?

### 💭 Это Value Object, если:

- ✅ Объект описывает характеристику чего-то?
- ✅ Важны только значения, а не идентичность?
- ✅ Объект можно безопасно заменить на другой с теми же значениями?
- ✅ Объект не изменяется после создания?

---

## 🎯 Best Practices

> [!success] Рекомендации для Entity
> 
> - Используйте `Guid` или `long` для ID
> - Инкапсулируйте бизнес-логику
> - Валидируйте изменения состояния
> - Реализуйте `Equals()` и `GetHashCode()` по ID

> [!success] Рекомендации для Value Object
> 
> - Делайте объекты неизменяемыми
> - Используйте `record` в C# 9+
> - Валидируйте в конструкторе
> - Реализуйте `IEquatable<T>`
> - Перегружайте операторы при необходимости

---

## 🔗 Связанные концепции

- [[Domain Services]]
- [[Aggregates]]
- [[Repositories]]
- [[Domain Events|Domain Events]]

---

> [!note] Заключение Правильное понимание различий между Entity и Value Object критически важно для создания чистой архитектуры. Entity представляют объекты с идентичностью и жизненным циклом, а Value Object описывают характеристики без собственной идентичности.

