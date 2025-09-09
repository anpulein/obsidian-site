---
aliases:
  - MOC - GoF - Repository
  - Repository
tags:
  - type/moc
  - status/active
  - area/architecture
  - concept/repository
  - design-pattern/reposiroty
  - tech/csharp
  - tech/asp-net
  - tech/ef-core
  - concept/ddd
  - concept/clean-architecture
title: MOC - GoF - Repository
linter-yaml-title-alias: MOC - GoF - Repository
date created: Tuesday, September 9th 2025, 5:06:05 am
date modified: Tuesday, September 9th 2025, 5:51:36 am
---

## 🏷️ Tags

#type/moc #status/active #area/architecture #concept/repository #design-pattern/reposiroty #tech/csharp #tech/asp-net #tech/ef-core #concept/ddd #concept/clean-architecture 

---

# MOC - GoF - Repository

> [!abstract] 📋 Что изучаем
> 
> - [ ]  Определение и назначение Repository Pattern
> - [ ]  Классическая реализация в .NET
> - [ ]  Интеграция с Entity Framework Core
> - [ ]  Generic Repository и его проблемы
> - [ ]  Критика паттерна и альтернативы
> - [ ]  Современные подходы в .NET

---

## 🎯 Определение

**Repository Pattern** — структурный паттерн проектирования, который инкапсулирует логику доступа к данным и обеспечивает более объектно-ориентированный взгляд на персистентный слой.

> [!tip] 💡 Ключевая идея Создать единый интерфейс для работы с данными, скрывая детали реализации хранилища (база данных, файлы, веб-сервис).

### Основные принципы

- **Разделение ответственности** — бизнес-логика отделена от логики доступа к данным
- **Тестируемость** — легко мокать для unit-тестов
- **Замещаемость** — можно менять источник данных без изменения бизнес-логики

---

## 🗺️ Карта содержания

### 📚 Основы паттерна

- [[GoF.Repository.BasicImplementation|Базовая реализация]] — классический подход с интерфейсами
- [[GoF.Repository.Interfaces&Contracts|Интерфейсы и контракты]] — проектирование API репозитория

### 🔧 Реализация с EF Core

- [[GoF.Repository.EF-Core|EF Core]] — интеграция с ORM
- [[GoF.Repository.DbContext|DbContext]] — когда EF Core сам является репозиторием

### 🎭 Продвинутые техники

- [[GoF.Repository.Generic|Generic]] — универсальный репозиторий и его подводные камни
- [[GoF.Repository.Specification|Specification]] — гибкие запросы с паттерном Спецификация
- [[GoF.Repository.UoW|Unit of Work]] — управление транзакциями

### 🔍 Критический анализ

- [[GoF.Repository.Problems|Критика и проблемы]] — почему паттерн не всегда нужен
- [[GoF.Repository.Alternatives|Альтернативы]] — современные подходы в .NET

### 💼 Практика

- [[Repository Pattern - Примеры кода]] — готовые snippets для копирования
- [[Repository Pattern - Best Practices]] — лучшие практики и рекомендации

---

## ⚡ Быстрый старт

csharp

```csharp
// Простейший интерфейс репозитория
public interface IUserRepository
{
    Task<User?> GetByIdAsync(int id);
    Task<IEnumerable<User>> GetAllAsync();
    Task AddAsync(User user);
    Task UpdateAsync(User user);
    Task DeleteAsync(int id);
}

// Базовая реализация
public class UserRepository : IUserRepository
{
    private readonly AppDbContext _context;
    
    public UserRepository(AppDbContext context)
    {
        _context = context;
    }
    
    public async Task<User?> GetByIdAsync(int id)
        => await _context.Users.FindAsync(id);
        
    // ... остальные методы
}
```

---

## 🤔 Когда использовать

> [!success] ✅ Используй Repository когда
> 
> - Нужна независимость от конкретной ORM
> - Сложная логика формирования запросов
> - Множественные источники данных
> - Строгие требования к тестируемости

> [!warning] ⚠️ Не используй когда
> 
> - Простое CRUD-приложение с EF Core
> - DbContext уже хорошо mockается в тестах
> - Нет планов менять источник данных
> - Команда не знакома с паттерном

---

## 🔗 Связанные концепции

- [[Unit of Work Pattern]] — управление транзакциями
- [[Specification Pattern]] — гибкие запросы
- [[DDD.Repository|Repository]] — репозиторий в контексте Domain-Driven Design
- [[Clean Architecture - Data Layer]] — место репозитория в чистой архитектуре

---

## 📊 Сравнение подходов

|Подход|Плюсы|Минусы|Когда использовать|
|---|---|---|---|
|**Классический Repository**|Простота, тестируемость|Много кода, дублирование|Простые CRUD операции|
|**Generic Repository**|Меньше кода|Потеря типобезопасности|Быстрое прототипирование|
|**DbContext как Repository**|Нет лишней абстракции|Зависимость от EF Core|Большинство .NET приложений|
|**Repository + Specification**|Гибкость запросов|Сложность|Сложные запросы, много фильтров|

---

> [!example] 🎯 Итог Repository Pattern — полезный инструмент, но не серебряная пуля. В современных .NET приложениях часто DbContext + правильная архитектура решают те же задачи проще и эффективнее.
