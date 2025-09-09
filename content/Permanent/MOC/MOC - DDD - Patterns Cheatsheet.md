---
title: MOC - DDD - Patterns Cheatsheet
aliases:
  - MOC - DDD - Patterns Cheatsheet
  - DDD - Patterns Cheatsheet
  - Patterns Cheatsheet
linter-yaml-title-alias: MOC - DDD - Patterns Cheatsheet
date created: Monday, September 8th 2025, 6:33:39 am
date modified: Monday, September 8th 2025, 2:22:05 pm
tags:
  - type/moc
  - concept/ddd
  - area/architecture
  - area/development
  - status/active
---
## 🏷️ Tags

#type/moc #concept/ddd #area/architecture #area/development #status/active 

---

# MOC - DDD - Patterns Cheatsheet

> [!info] 📋 О заметке Центральный справочник по паттернам Domain-Driven Design с быстрым доступом к ключевым концепциям и их практическому применению

---

## ✅ Что будет раскрыто

- [ ] **Strategic Patterns** — стратегические паттерны для моделирования доменов
- [ ] **Tactical Patterns** — тактические паттерны для реализации
- [ ] **Integration Patterns** — паттерны интеграции между контекстами
- [ ] **Infrastructure Patterns** — инфраструктурные паттерны
- [ ] **Anti-Patterns** — что следует избегать
- [ ] **Quick Reference** — быстрая справка по применению
- [ ] **Code Examples** — примеры реализации

---

## 📑 Содержание

### 🎯 Strategic Design Patterns

### ⚙️ Tactical Design Patterns

### 🔗 Integration Patterns

### 🏗️ Infrastructure Patterns

### ⚠️ Anti-Patterns & Pitfalls

### 📖 Quick Reference Guide

---

## 🎯 Strategic Design Patterns

> [!tip] 💡 Стратегические паттерны Помогают структурировать домен на высоком уровне и определить границы между различными частями системы

### 📊 Обзор стратегических паттернов

| Паттерн                                              | Цель                                     | Когда использовать                  |
| ---------------------------------------------------- | ---------------------------------------- | ----------------------------------- |
| **[[MOC - DDD - Bounded Context\|Bounded Context]]** | Явное определение границ модели          | Всегда - основа DDD                 |
| **[[DDD.SubdomainTypes\|🎯 Subdomain Types]]**    | Разделение бизнес-области                | При сложной предметной области      |
| **[[DDD.Context Map\|Context Map]]**               | Визуализация отношений между контекстами | При множественных командах/системах |
| **[[Ubiquitous Language]]**                          | Единый язык команды                      | Всегда - основа коммуникации        |

### 🗺️ Context Relationships

```mermaid
graph TB
    subgraph "Strategic Patterns"
        BC1[Bounded Context 1]
        BC2[Bounded Context 2]
        BC3[Bounded Context 3]
        
        BC1 ---|Partnership| BC2
        BC2 ---|Customer/Supplier| BC3
        BC1 -.->|Anti-Corruption Layer| BC3
    end
```

---

## ⚙️ Tactical Design Patterns

> [!gear] 🔧 Тактические паттерны Конкретные строительные блоки для реализации доменной модели

### 📋 Building Blocks

| Паттерн                                      | Ответственность             | Характеристики                     |
| -------------------------------------------- | --------------------------- | ---------------------------------- |
| **[[DDD.Entity\|Entity]]**                 | Объекты с идентичностью     | Изменяемые, уникальный ID          |
| **[[DDD.ValueObject\|Value Object]]**     | Описательные объекты        | Неизменяемые, без ID               |
| **[[DDD.Aggregate\|Aggregate]]**           | Границы консистентности     | Корень + внутренние объекты        |
| **[[DDD.Domain Service\|Domain Service]]** | Бизнес-логика без состояния | Операции над несколькими объектами |
| **[[DDD.Repository\|Repository]]**         | Доступ к агрегатам          | Инкапсуляция хранилища             |
| **[[Factory]]**                              | Создание сложных объектов   | Скрытие логики конструирования     |

### 🏗️ Tactical Patterns Structure

```mermaid
classDiagram
    class Aggregate {
        +AggregateRoot
        +Entities[]
        +ValueObjects[]
        +DomainEvents[]
    }
    
    class Entity {
        +ID
        +BusinessLogic()
    }
    
    class ValueObject {
        +Properties
        +Equals()
        +GetHashCode()
    }
    
    class DomainService {
        +BusinessOperation()
    }
    
    class Repository {
        +Save(Aggregate)
        +FindById(ID)
        +FindByCriteria()
    }
    
    Aggregate *-- Entity
    Aggregate *-- ValueObject
    DomainService ..> Aggregate
    Repository ..> Aggregate
```

---

## 🔗 Integration Patterns

> [!link] 🌐 Интеграционные паттерны Способы взаимодействия между Bounded Context'ами

### 🔄 Context Integration

| Паттерн                                                        | Описание                   | Плюсы                | Минусы                   |
| -------------------------------------------------------------- | -------------------------- | -------------------- | ------------------------ |
| **[[Shared Kernel]]**                                          | Общая модель               | Простота             | Сильная связанность      |
| **[[Customer-Supplier]]**                                      | Отношения поставщик-клиент | Четкие обязательства | Зависимость              |
| **[[Conformist]]**                                             | Принятие чужой модели      | Быстрая интеграция   | Потеря автономии         |
| **[[DDD.Anti-CorruptionLayer\|🛡️ Anti-Corruption Layer]]** | Защитный слой              | Изоляция изменений   | Дополнительная сложность |
| **[[Open Host Service]]**                                      | Публичный API              | Стандартизация       | Необходимость поддержки  |
| **[[Published Language]]**                                     | Общий формат данных        | Универсальность      | Сложность эволюции       |

### 🛡️ Anti-Corruption Layer Pattern

```mermaid
graph LR
    subgraph "Your Context"
        YM[Your Model]
        ACL[Anti-Corruption Layer]
        YM <--> ACL
    end
    
    subgraph "External Context"  
        EM[External Model]
        API[External API]
        EM <--> API
    end
    
    ACL <--> API
    
    ACL -.-> T[Translator]
    ACL -.-> A[Adapter]
    ACL -.-> F[Facade]
```

---

## 🏗️ Infrastructure Patterns

> [!wrench] 🔧 Инфраструктурные паттерны Технические паттерны для поддержки доменной модели

### 📦 Implementation Patterns

|Паттерн|Назначение|Реализация|
|---|---|---|
|**[[Domain Events]]**|Уведомления о бизнес-событиях|Event-driven архитектура|
|**[[Specification]]**|Бизнес-правила как объекты|Композиция условий|
|**[[Unit of Work]]**|Управление транзакциями|Отслеживание изменений|
|**[[Data Transfer Object]]**|Передача данных между слоями|Сериализуемые структуры|

### 🎭 Layered Architecture

```mermaid
graph TB
    subgraph "DDD Layers"
        UI[Presentation Layer]
        APP[Application Layer]
        DOM[Domain Layer]
        INF[Infrastructure Layer]
        
        UI --> APP
        APP --> DOM
        APP --> INF
        INF --> DOM
    end
    
    subgraph "Domain Layer Details"
        ENT[Entities]
        VO[Value Objects]
        AGG[Aggregates]
        DS[Domain Services]
        REP[Repository Interfaces]
        
        DOM --> ENT
        DOM --> VO
        DOM --> AGG
        DOM --> DS
        DOM --> REP
    end
```

---

## ⚠️ Anti-Patterns & Pitfalls

> [!warning] 🚫 Что избегать Распространенные ошибки при применении DDD

### 💀 Common Anti-Patterns

|Anti-Pattern|Описание|Почему плохо|Решение|
|---|---|---|---|
|**Anemic Domain Model**|Модель без поведения|Нарушение инкапсуляции|Перенести логику в Entity|
|**Big Ball of Mud**|Отсутствие границ|Сложность сопровождения|Выделить Bounded Context|
|**God Object**|Один объект делает все|Нарушение SRP|Декомпозиция|
|**Primitive Obsession**|Примитивы вместо VO|Потеря семантики|Создать Value Objects|

### 🔍 Code Smells в DDD

```csharp
// ❌ Anti-Pattern: Anemic Domain Model
public class User
{
    public string Email { get; set; }
    public string Password { get; set; }
    public bool IsActive { get; set; }
}

public class UserService 
{
    public void ActivateUser(User user) 
    {
        user.IsActive = true; // Логика в сервисе
    }
}

// ✅ Rich Domain Model  
public class User
{
    public Email Email { get; private set; }
    public Password Password { get; private set; }
    public bool IsActive { get; private set; }
    
    public void Activate() 
    {
        IsActive = true; // Логика в модели
        // Бизнес-правила активации
    }
}
```

---

## 📖 Quick Reference Guide

> [!reference] 📚 Быстрая справка Когда использовать какой паттерн

### 🎯 Decision Matrix

```mermaid
flowchart TD
    Start([Проектирую систему]) --> Strategic{Стратегический уровень?}
    
    Strategic -->|Да| BC[Определить Bounded Contexts]
    BC --> CM[Создать Context Map]
    CM --> UL[Установить Ubiquitous Language]
    
    Strategic -->|Нет| Tactical{Тактический уровень?}
    Tactical -->|Да| Identity{Есть идентичность?}
    Identity -->|Да| Entity[Entity]
    Identity -->|Нет| VO[Value Object]
    
    Entity --> Behavior{Сложное поведение?}
    Behavior -->|Да| Agg[Aggregate]
    Behavior -->|Нет| Simple[Simple Entity]
    
    Tactical -->|Нет| Integration{Интеграция?}
    Integration -->|Да| Trust{Доверяю внешней модели?}
    Trust -->|Да| Conform[Conformist]
    Trust -->|Нет| ACL[Anti-Corruption Layer]
```

### 📋 Pattern Selection Checklist

- [ ] **Bounded Context** - всегда первый шаг
- [ ] **Aggregate** - для группировки связанных Entity
- [ ] **Value Object** - для описательных характеристик
- [ ] **Domain Service** - для операций над несколькими Aggregates
- [ ] **Repository** - для доступа к данным
- [ ] **Anti-Corruption Layer** - при интеграции с legacy системами

---

## 🔗 Связанные заметки

### 📚 Детальное изучение

- [[DDD Strategic Design]] - стратегическое проектирование
- [[DDD Tactical Patterns]] - тактические паттерны
- [[DDD Implementation Examples]] - примеры реализации
- [[DDD Best Practices]] - лучшие практики

### 🛠️ Практическое применение

- [[DDD with .NET]] - реализация на .NET
- [[DDD Testing Strategies]] - стратегии тестирования
- [[DDD Migration Guide]] - миграция к DDD

### 🔄 Связанные концепции

- [[Clean Architecture]] - архитектурные принципы
- [[CQRS]] - разделение команд и запросов
- [[ArchPat.EventSourcing]] - событийное хранилище
- [[Microservices]] - микросервисная архитектура

---

> [!tip] 💡 Следующие шаги
> 
> 1. Изучите [[MOC - DDD - Bounded Context|Bounded Context]] как основу DDD
> 2. Освойте [[Aggregate Pattern]] для моделирования
> 3. Примените [[Repository Pattern]] для персистентности
> 4. Рассмотрите [[DDD.Anti-CorruptionLayer]] для интеграций