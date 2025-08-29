---
title: .NET init - Полное руководство
aliases:
  - .NET init - Полное руководство
linter-yaml-title-alias: .NET init - Полное руководство
date created: Wednesday, August 13th 2025, 11:20:37 am
date modified: Wednesday, August 13th 2025, 11:24:05 am
tags:
  - type/area
  - area/development
  - tech/csharp
  - concept/immutable
  - impl/init
---
# .NET init - Полное руководство

> [!info] Определение **init** - это ключевое слово в C#, введенное в версии 9.0, которое позволяет создавать свойства, доступные для записи только во время инициализации объекта.

## 🎯 Основные концепции

### Что такое init?

`init` - это специальный accessor (аксессор) для свойств, который работает следующим образом:

- ✅ **Можно** установить значение во время создания объекта
- ✅ **Можно** установить значение в конструкторе
- ❌ **Нельзя** изменить значение после создания объекта

---

## 🔍 Синтаксис

```csharp
public class Person
{
    public string Name { get; init; }
    public int Age { get; init; }
}
```

> [!tip] Сравнение
> 
> - `{ get; set; }` - можно читать и изменять всегда
> - `{ get; init; }` - можно читать всегда, изменять только при инициализации
> - `{ get; }` - только для чтения (readonly)

---

## 📊 Практические примеры

### Пример 1: Базовое использование

```csharp
public class Book
{
    public string Title { get; init; }
    public string Author { get; init; }
    public int Pages { get; init; }
}

// Использование:
var book = new Book
{
    Title = "Война и мир",
    Author = "Л.Н. Толстой",
    Pages = 1300
};

Console.WriteLine($"{book.Title} - {book.Author}");

// ❌ Это вызовет ошибку компиляции:
// book.Title = "Новое название"; // Ошибка!
```

### Пример 2: init в конструкторе

```csharp
public class Car
{
    public string Brand { get; init; }
    public string Model { get; init; }
    public int Year { get; init; }
    
    public Car(string brand, string model, int year)
    {
        Brand = brand;   // ✅ Работает в конструкторе
        Model = model;   // ✅ Работает в конструкторе
        Year = year;     // ✅ Работает в конструкторе
    }
}

var car = new Car("Toyota", "Camry", 2023);
```

### Пример 3: Комбинирование с другими модификаторами

```csharp
public class Product
{
    public string Name { get; init; }
    public decimal Price { get; private set; }
    public string Category { get; init; }
    public DateTime CreatedAt { get; } = DateTime.Now;
    
    public void UpdatePrice(decimal newPrice)
    {
        Price = newPrice; // ✅ private set позволяет изменять внутри класса
    }
}
```

---

## 🆚 Сравнение с другими подходами

|Подход|Изменяемость|Инициализация|Производительность|
|---|---|---|---|
|`{ get; set; }`|В любое время|Гибкая|Хорошая|
|`{ get; init; }`|Только при создании|Гибкая|Отличная|
|`readonly поля`|Никогда|Только в конструкторе|Отличная|
|`{ get; }`|Никогда|Только в объявлении|Отличная|

---

## 🏗️ Records и init

> [!example] Records автоматически используют init В C# 9.0 также были введены **records**, которые автоматически генерируют init-свойства:

```csharp
// Record с init свойствами
public record Person(string Name, int Age);

// Эквивалентно:
public record Person
{
    public string Name { get; init; }
    public int Age { get; init; }
}

// Использование:
var person1 = new Person("Иван", 25);
var person2 = person1 with { Age = 26 }; // Создание копии с изменением
```

---

## 🔧 Продвинутые сценарии

### Пример 4: init с валидацией

```csharp
public class Email
{
    private string _address;
    
    public string Address 
    { 
        get => _address;
        init
        {
            if (string.IsNullOrWhiteSpace(value) || !value.Contains("@"))
                throw new ArgumentException("Некорректный email");
            _address = value;
        }
    }
}

// Использование:
var email = new Email { Address = "user@example.com" }; // ✅ OK
// var badEmail = new Email { Address = "invalid" }; // ❌ Exception
```

### Пример 5: Наследование и init

```csharp
public class Animal
{
    public string Name { get; init; }
    public string Species { get; init; }
}

public class Dog : Animal
{
    public string Breed { get; init; }
    
    public Dog()
    {
        Species = "Собака"; // ✅ Можно устанавливать в конструкторе
    }
}

var dog = new Dog 
{ 
    Name = "Бобик",
    Breed = "Лабрадор"
};
```

---

## ⚠️ Важные особенности и ограничения

> [!warning] Ограничения
> 
> 1. **init** доступен только в C# 9.0 и выше
> 2. Нельзя использовать `init` и `set` одновременно
> 3. `init` свойства нельзя изменять после создания объекта

> [!danger] Распространенные ошибки
> 
> ```csharp
> var person = new Person { Name = "Иван" };
> person.Name = "Петр"; // ❌ Ошибка компиляции!
> 
> // Правильно - создать новый объект:
> var newPerson = new Person { Name = "Петр" };
> ```

---

## 🎯 Когда использовать init?

### ✅ Хорошо подходит для:

- **Неизменяемых данных** (immutable objects)
- **Конфигурационных объектов**
- **DTO (Data Transfer Objects)**
- **Value Objects**
- **Модели данных**

```csharp
// Конфигурация
public class DatabaseConfig
{
    public string ConnectionString { get; init; }
    public int Timeout { get; init; }
    public bool EnableRetries { get; init; }
}

// DTO
public class UserDto
{
    public int Id { get; init; }
    public string Username { get; init; }
    public string Email { get; init; }
}
```

### ❌ Не подходит для:

- **Объектов с изменяемым состоянием**
- **Кэшируемых значений**
- **Счетчиков и метрик**

---

## 🔍 Под капотом

> [!info] Технические детали Компилятор C# генерирует специальный метод `init` accessor, который работает аналогично `set`, но с проверкой контекста вызова.

```csharp
// Что видим мы:
public string Name { get; init; }

// Что генерирует компилятор (упрощенно):
private string <Name>k__BackingField;
public string get_Name() => <Name>k__BackingField;
public void init_Name(string value) => <Name>k__BackingField = value;
```

---

## 📚 Заключение

> [!success] Ключевые преимущества init:
> 
> - 🔒 **Безопасность** - защита от случайных изменений
> - 🚀 **Производительность** - оптимизации компилятора
> - 📖 **Читаемость** - ясность намерений в коде
> - 🛡️ **Неизменяемость** - поддержка функционального стиля

**init** - это мощный инструмент для создания безопасных и производительных неизменяемых объектов в C#. Используйте его для создания надежных моделей данных и конфигураций!
