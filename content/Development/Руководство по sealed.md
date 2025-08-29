---
title: Ключевое слово `sealed` в C#
aliases:
  - Ключевое слово `sealed` в C#
linter-yaml-title-alias: Ключевое слово `sealed` в C#
date created: Friday, August 8th 2025, 10:50:51 am
date modified: Wednesday, August 13th 2025, 11:03:08 am
tags:
  - type/area
  - area/development
  - tech/csharp
  - concept/immutable
  - "impl/sealed"
---
# Ключевое слово `sealed` в C#

> [!info] Определение `sealed` - это модификатор в C#, который **запрещает наследование** от класса или переопределение виртуальных методов/свойств.

---

## 📋 Содержание

- [[#Основные концепции]]
- [[#sealed классы]]
- [[#sealed методы и свойства]]
- [[#Практические примеры]]
- [[#Преимущества и недостатки]]
- [[#Лучшие практики]]

---

## 🎯 Основные концепции

### Что такое `sealed`?

`sealed` может применяться к:

- **Классам** - запрещает создание наследников
- **Методам** - запрещает переопределение в дочерних классах
- **Свойствам** - запрещает переопределение в дочерних классах

> [!warning] Важно `sealed` методы и свойства могут использоваться **только** в связке с `override`!

---

## 🏗️ sealed классы

### Базовый синтаксис

```csharp
public sealed class MyClass
{
    // Реализация класса
}
```

### Пример sealed класса

```csharp
// Базовый класс
public class Animal
{
    public virtual void MakeSound()
    {
        Console.WriteLine("Животное издает звук");
    }
}

// sealed класс - финальный в иерархии наследования
public sealed class Cat : Animal
{
    public override void MakeSound()
    {
        Console.WriteLine("Мяу!");
    }
    
    public void Purr()
    {
        Console.WriteLine("Мурр-мурр");
    }
}

// ❌ Это вызовет ошибку компиляции!
// public class PersianCat : Cat { } // Ошибка: Cannot derive from sealed type 'Cat'
```

> [!example] Примеры sealed классов в .NET
> 
> - `string` - запечатанный класс для работы со строками
> - `DateTime` - структура времени и даты
> - `Decimal` - структура для точных вычислений

---

## ⚙️ sealed методы и свойства

### Синтаксис

```csharp
public sealed override ВозвращаемыйТип ИмяМетода()
{
    // Реализация
}
```

### Пример иерархии с sealed методами

```csharp
// Базовый класс
public class Vehicle
{
    public virtual void Start()
    {
        Console.WriteLine("Транспорт запускается");
    }
    
    public virtual void Stop()
    {
        Console.WriteLine("Транспорт останавливается");
    }
}

// Промежуточный класс
public class Car : Vehicle
{
    // Обычное переопределение
    public override void Start()
    {
        Console.WriteLine("Автомобиль заводится с ключа");
    }
    
    // sealed переопределение - дальше переопределить нельзя!
    public sealed override void Stop()
    {
        Console.WriteLine("Автомобиль тормозит");
    }
}

// Наследник Car
public class SportsCar : Car
{
    // ✅ Можно переопределить Start (не sealed)
    public override void Start()
    {
        Console.WriteLine("Спорткар ревет двигателем!");
    }
    
    // ❌ Нельзя переопределить Stop (sealed в Car)
    // public override void Stop() { } // Ошибка компиляции!
}
```

---

## 🔍 Практические примеры

### Пример 1: Система логирования

```csharp
public abstract class Logger
{
    public abstract void Log(string message);
    
    public virtual void LogError(string error)
    {
        Log($"ERROR: {error}");
    }
}

public class FileLogger : Logger
{
    public override void Log(string message)
    {
        File.AppendAllText("log.txt", $"{DateTime.Now}: {message}\n");
    }
    
    // Запечатываем метод - никто не может изменить формат ошибок
    public sealed override void LogError(string error)
    {
        Log($"🔥 CRITICAL ERROR: {error.ToUpper()}");
    }
}

// Специализированный логгер
public class DatabaseLogger : FileLogger
{
    public override void Log(string message)
    {
        // Сохраняем в базу данных
        SaveToDatabase(message);
    }
    
    // ❌ Нельзя переопределить LogError - он sealed!
    // public override void LogError(string error) { }
    
    private void SaveToDatabase(string message)
    {
        Console.WriteLine($"Сохранено в БД: {message}");
    }
}
```

### Пример 2: Система безопасности

```csharp
// Базовый класс для всех пользователей
public class User
{
    public virtual bool CanAccess(string resource)
    {
        return false; // По умолчанию доступа нет
    }
}

// sealed class - администратор не может иметь наследников
public sealed class Administrator : User
{
    public override bool CanAccess(string resource)
    {
        return true; // Админ имеет доступ ко всему
    }
    
    public void ManageUsers()
    {
        Console.WriteLine("Управление пользователями");
    }
}

// Обычный пользователь с ограничениями
public class RegularUser : User
{
    protected string[] allowedResources;
    
    public RegularUser(string[] resources)
    {
        allowedResources = resources;
    }
    
    // sealed метод - логика проверки не может быть изменена
    public sealed override bool CanAccess(string resource)
    {
        return allowedResources.Contains(resource);
    }
}

// Гостевой пользователь
public class GuestUser : RegularUser
{
    public GuestUser() : base(new[] { "public-content" })
    {
    }
    
    // ❌ Нельзя изменить логику доступа - она sealed!
    // public override bool CanAccess(string resource) { }
}
```

---

## ⚖️ Преимущества и недостатки

### ✅ Преимущества

|Преимущество|Описание|
|---|---|
|**Безопасность**|Предотвращает нежелательные изменения в критичной логике|
|**Производительность**|Компилятор может применить больше оптимизаций|
|**Стабильность API**|Гарантирует неизменность поведения ключевых компонентов|
|**Целостность дизайна**|Сохраняет изначально задуманную архитектуру|

### ❌ Недостатки

|Недостаток|Описание|
|---|---|
|**Ограниченная расширяемость**|Нельзя адаптировать под новые требования|
|**Сложности в тестировании**|Нельзя создать mock-объекты через наследование|
|**Жесткость архитектуры**|Может препятствовать будущим изменениям|

---

## 🎯 Лучшие практики

### ✅ Когда использовать `sealed`

> [!tip] Используйте sealed, когда:
> 
> - Класс представляет **неизменяемую концепцию** (как `string`)
> - Нужно **защитить критичную логику безопасности**
> - Класс служит **конечной реализацией** определенной функциональности
> - Требуется **максимальная производительность**

### ❌ Когда НЕ использовать `sealed`

> [!warning] Избегайте sealed, когда:
> 
> - Планируется **расширение функциональности** в будущем
> - Класс может потребовать **различных реализаций**
> - Нужна **гибкость для тестирования**
> - Разрабатывается **публичная библиотека**

---

## 🔬 Альтернативы sealed

### Композиция вместо наследования

```csharp
// Вместо sealed класса можно использовать композицию
public interface ISecurityValidator
{
    bool ValidateAccess(string token);
}

public class TokenValidator : ISecurityValidator
{
    public bool ValidateAccess(string token)
    {
        // Логика валидации
        return !string.IsNullOrEmpty(token);
    }
}

public class SecureService
{
    private readonly ISecurityValidator _validator;
    
    public SecureService(ISecurityValidator validator)
    {
        _validator = validator;
    }
    
    public void DoSecureOperation(string token)
    {
        if (_validator.ValidateAccess(token))
        {
            Console.WriteLine("Операция выполнена безопасно");
        }
    }
}
```

---

## 📝 Заключение

> [!summary] Ключевые моменты
> 
> - `sealed` **предотвращает наследование** классов и переопределение методов
> - Используется для **защиты критичной логики** и **оптимизации производительности**
> - Должен применяться **осознанно** с учетом будущих требований
> - В .NET многие базовые типы являются sealed (`string`, `DateTime`, и др.)

---

## 🔗 Связанные концепции

- [[virtual и override в C#]]
- [[abstract классы и методы]]
- [[Принципы SOLID]]
- [[Паттерн Strategy]]