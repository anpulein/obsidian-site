---
title: MOC - Archeticture Patterns - Hexagonal Architecture
aliases:
  - MOC - Archeticture Patterns - Hexagonal Architecture
  - Hexagonal Architecture
linter-yaml-title-alias: MOC - Archeticture Patterns - Hexagonal Architecture
date created: Monday, September 8th 2025, 7:13:22 am
date modified: Tuesday, September 9th 2025, 5:10:51 am
tags:
  - type/moc
  - area/architecture
  - area/development
  - concept/ddd
  - status/active
  - concept/hexagonal-architecture
  - concept/ports-adapters
  - concept/clean-architecture
  - concept/dependency-inversion
  - tech/csharp
  - tech/asp-net
  - design-pattern/depe
---
## 🏷️ Tags

#type/moc #concept/hexagonal-architecture #concept/ports-adapters #concept/clean-architecture #concept/dependency-inversion #area/architecture #tech/csharp #tech/asp-net #design-pattern/depe #status/active 

---

# MOC - Archeticture Patterns - Hexagonal Architecture

> [!abstract] 📋 Чек-лист изучения
> 
> - [ ] Понять основные принципы и философию паттерна
> - [ ] Изучить структуру: домен, порты, адаптеры
> - [ ] Освоить реализацию портов и адаптеров в .NET
> - [ ] Настроить DI контейнер для Hexagonal Architecture
> - [ ] Применить на практике с примерами кода
> - [ ] Изучить тестирование архитектуры
> - [ ] Понять отличия от других архитектурных паттернов

---

## 🎯 Что такое Hexagonal Architecture

**Hexagonal Architecture** (архитектура портов и адаптеров) — архитектурный паттерн, предложенный Алистером Кокберном, который изолирует бизнес-логику приложения от внешних зависимостей через систему **портов** и **адаптеров**.

> [!tip] ✨ Ключевая идея Приложение должно быть одинаково управляемо пользователями, программами, автоматизированными тестами или пакетными скриптами, а также разрабатываться и тестироваться в изоляции от внешних устройств и баз данных.

### 🔗 Основные принципы

1. **Изоляция домена** — бизнес-логика не зависит от внешних систем
2. **Обратная зависимость** — инфраструктура зависит от домена, а не наоборот
3. **Тестируемость** — домен можно тестировать без внешних зависимостей
4. **Гибкость** — легкая замена внешних систем без изменения домена

---

## 📚 Содержание

### 🏛️ Архитектурные основы

- [[Hexagonal Architecture - Структура и компоненты]] — детальный разбор архитектуры
- [[Hexagonal Architecture - Порты]] — интерфейсы взаимодействия с доменом
- [[Hexagonal Architecture - Адаптеры]] — реализации портов для внешних систем

### ⚙️ Реализация в .NET

- [[Hexagonal Architecture - Настройка проекта .NET]] — структура решения и проектов
- [[Hexagonal Architecture - Dependency Injection]] — настройка DI контейнера
- [[Hexagonal Architecture - Примеры кода]] — практические примеры реализации

### 🧪 Тестирование и качество

- [[Hexagonal Architecture - Тестирование]] — стратегии тестирования архитектуры
- [[Hexagonal Architecture - Мониторинг и логирование]] — наблюдаемость в гексагональной архитектуре

---

## 🏗️ Схема архитектуры

```mermaid
graph TB
    subgraph "🌐 External World"
        UI[Web UI]
        API[REST API]
        DB[(Database)]
        Queue[Message Queue]
        Email[Email Service]
    end
    
    subgraph "🔌 Adapters Layer"
        WebAdapter[Web Adapter]
        ApiAdapter[API Adapter]
        DbAdapter[DB Adapter]
        QueueAdapter[Queue Adapter]
        EmailAdapter[Email Adapter]
    end
    
    subgraph "🚪 Ports Layer"
        InPort[Input Ports]
        OutPort[Output Ports]
    end
    
    subgraph "🎯 Domain Core"
        Domain[Business Logic]
        Entities[Domain Entities]
        Services[Domain Services]
        Rules[Business Rules]
    end
    
    UI --> WebAdapter
    API --> ApiAdapter
    WebAdapter --> InPort
    ApiAdapter --> InPort
    
    InPort --> Domain
    Domain --> Services
    Services --> Entities
    Services --> Rules
    
    Domain --> OutPort
    OutPort --> DbAdapter
    OutPort --> QueueAdapter  
    OutPort --> EmailAdapter
    
    DbAdapter --> DB
    QueueAdapter --> Queue
    EmailAdapter --> Email
    
    classDef external fill:#e1f5fe
    classDef adapter fill:#f3e5f5
    classDef port fill:#e8f5e8
    classDef domain fill:#fff3e0
    
    class UI,API,DB,Queue,Email external
    class WebAdapter,ApiAdapter,DbAdapter,QueueAdapter,EmailAdapter adapter
    class InPort,OutPort port
    class Domain,Entities,Services,Rules domain
```

---

## 🆚 Сравнение с другими архитектурами

|Аспект|Hexagonal|Clean Architecture|N-Layer|
|---|---|---|---|
|**Зависимости**|Все направлены к домену|Все направлены к домену|Сверху вниз|
|**Тестируемость**|Высокая|Высокая|Средняя|
|**Сложность**|Средняя|Высокая|Низкая|
|**Гибкость**|Высокая|Высокая|Низкая|

---

## 🛠️ Быстрый старт

> [!example] 📝 Минимальная реализация
> 
> 1. **Создай структуру проекта**:
>     
>     ```
>     Solution/
>     ├── Domain/           # Бизнес-логика
>     ├── Application/      # Порты и use cases
>     ├── Infrastructure/   # Адаптеры
>     └── Presentation/     # Web API/UI
>     ```
>     
> 2. **Определи порты** (интерфейсы в Application)
>     
> 3. **Реализуй адаптеры** (в Infrastructure)
>     
> 4. **Настрой DI** для связывания портов и адаптеров
>     

---

## 📖 Связанные концепции

- [[Концепции (concept)#concept/dependency-inversion|Dependency Inversion Principle]]
- [[MOC - Clean Architcture|Clean Architecture]]
- [[DDD.Domain Service|DDD - Domain Service]]
- [[Dependency Injection]]
- [[SOLID Principles]]

---

> [!summary] 🎯 Итоги Hexagonal Architecture — мощный паттерн для создания гибких, тестируемых и независимых от инфраструктуры приложений. Особенно эффективен в .NET экосистеме благодаря встроенной поддержке DI и богатым возможностям тестирования.