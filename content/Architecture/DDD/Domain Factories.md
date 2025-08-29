---
title: Domain Factories
aliases:
  - Domain Factories
linter-yaml-title-alias: Domain Factories
date created: Friday, August 22nd 2025, 6:12:58 am
date modified: Friday, August 22nd 2025, 7:52:06 am
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

> [!info] 📋 **Содержание**
> 
> - [[#Что такое Domain Factory?|Что такое Domain Factory?]]
> - [[#Зачем нужны Factory?|Зачем нужны Factory?]]
> - [[#Типы Factory|Типы Factory]]
> - [[#Примеры реализации|Примеры реализации]]
> - [[#Паттерны и лучшие практики|Паттерны и лучшие практики]]

---

## Что такое Domain Factory?

> [!note] 🏭 **Определение** **Domain Factory** — это паттерн в DDD, который инкапсулирует сложную логику создания доменных объектов, обеспечивая их корректную инициализацию и соблюдение бизнес-правил.

### Ключевые характеристики

- 🔧 **Инкапсулирует** сложность создания объектов
- ✅ **Гарантирует** валидность создаваемых объектов
- 🎯 **Централизует** логику конструирования
- 🔒 **Обеспечивает** соблюдение инвариантов

---

## Зачем нужны Factory?

> [!warning] ⚠️ **Проблемы без Factory**
> 
> ```csharp
> // ❌ Плохо: логика создания размазана по коду
> var user = new User();
> user.Id = Guid.NewGuid();
> user.Email = email;
> user.Status = UserStatus.Pending;
> user.CreatedAt = DateTime.UtcNow;
> // А что если забыли установить какое-то поле?
> ```

> [!success] ✅ **Решение с Factory**
> 
> ```csharp
> // ✅ Хорошо: вся логика в одном месте
> var user = UserFactory.CreatePendingUser(email);
> ```

### Преимущества Factory

|Преимущество|Описание|
|---|---|
|🛡️ **Безопасность**|Гарантия корректного создания|
|🧹 **Чистота кода**|Убирает дублирование|
|🔄 **Переиспользование**|Единая точка создания|
|🧪 **Тестируемость**|Легко мокать и тестировать|

---

## Типы Factory

### 1. Static Factory Methods

> [!example] 📝 **Простейший вариант**
> 
> ```csharp
> public class User
> {
>     private User(string email, UserStatus status)
>     {
>         Id = Guid.NewGuid();
>         Email = email;
>         Status = status;
>         CreatedAt = DateTime.UtcNow;
>     }
> 
>     // Factory методы
>     public static User CreatePendingUser(string email)
>         => new User(email, UserStatus.Pending);
> 
>     public static User CreateActiveUser(string email)
>         => new User(email, UserStatus.Active);
> 
>     public Guid Id { get; private set; }
>     public string Email { get; private set; }
>     public UserStatus Status { get; private set; }
>     public DateTime CreatedAt { get; private set; }
> }
> ```

### 2. Dedicated Factory Classes

> [!example] 🏗️ **Отдельный класс Factory**
> 
> ```csharp
> public interface IOrderFactory
> {
>     Order CreateOrder(CustomerId customerId, IEnumerable<OrderItem> items);
> }
> 
> public class OrderFactory : IOrderFactory
> {
>     private readonly IPricingService _pricingService;
>     private readonly IInventoryService _inventoryService;
> 
>     public OrderFactory(
>         IPricingService pricingService,
>         IInventoryService inventoryService)
>     {
>         _pricingService = pricingService;
>         _inventoryService = inventoryService;
>     }
> 
>     public Order CreateOrder(CustomerId customerId, IEnumerable<OrderItem> items)
>     {
>         // Проверка доступности товаров
>         foreach (var item in items)
>         {
>             if (!_inventoryService.IsAvailable(item.ProductId, item.Quantity))
>                 throw new ProductNotAvailableException(item.ProductId);
>         }
> 
>         // Расчет цены
>         var totalPrice = _pricingService.CalculateTotal(items);
> 
>         return new Order(
>             OrderId.NewId(),
>             customerId,
>             items.ToList(),
>             totalPrice,
>             DateTime.UtcNow);
>     }
> }
> ```

---

## Примеры реализации

### Factory для Complex Aggregate

> [!example] 🏢 **Создание сложного агрегата**
> 
> ```csharp
> public class ProjectFactory
> {
>     public static Project CreateProject(
>         string name,
>         ProjectManagerId managerId,
>         DateTime deadline,
>         Budget initialBudget)
>     {
>         // Валидация бизнес-правил
>         if (deadline <= DateTime.UtcNow)
>             throw new InvalidOperationException("Deadline должен быть в будущем");
> 
>         if (initialBudget.Amount <= 0)
>             throw new ArgumentException("Бюджет должен быть положительным");
> 
>         var project = new Project(
>             ProjectId.NewId(),
>             name,
>             managerId,
>             deadline,
>             initialBudget);
> 
>         // Добавление начальных задач
>         project.AddTask("Планирование проекта", managerId, deadline.AddDays(-7));
>         
>         return project;
>     }
> }
> ```

### Factory с внешними зависимостями

> [!example] 🔗 **Factory с сервисами**
> 
> ```csharp
> public class InvoiceFactory
> {
>     private readonly ITaxCalculator _taxCalculator;
>     private readonly INumberGenerator _numberGenerator;
> 
>     public InvoiceFactory(
>         ITaxCalculator taxCalculator,
>         INumberGenerator numberGenerator)
>     {
>         _taxCalculator = taxCalculator;
>         _numberGenerator = numberGenerator;
>     }
> 
>     public Invoice CreateInvoice(
>         CustomerId customerId,
>         IEnumerable<InvoiceItem> items,
>         Address billingAddress)
>     {
>         var invoiceNumber = _numberGenerator.GenerateInvoiceNumber();
>         var subtotal = items.Sum(x => x.Amount);
>         var tax = _taxCalculator.CalculateTax(subtotal, billingAddress);
> 
>         return new Invoice(
>             InvoiceId.NewId(),
>             invoiceNumber,
>             customerId,
>             items.ToList(),
>             subtotal,
>             tax,
>             billingAddress,
>             DateTime.UtcNow);
>     }
> }
> ```

---

## Паттерны и лучшие практики

### 1. Builder Pattern + Factory

> [!tip] 🔨 **Комбинированный подход**
> 
> ```csharp
> public class UserBuilder
> {
>     private string _email;
>     private string _firstName;
>     private string _lastName;
>     private List<Role> _roles = new();
> 
>     public UserBuilder WithEmail(string email)
>     {
>         _email = email;
>         return this;
>     }
> 
>     public UserBuilder WithName(string firstName, string lastName)
>     {
>         _firstName = firstName;
>         _lastName = lastName;
>         return this;
>     }
> 
>     public UserBuilder WithRole(Role role)
>     {
>         _roles.Add(role);
>         return this;
>     }
> 
>     public User Build()
>     {
>         ValidateRequiredFields();
>         
>         return new User(
>             UserId.NewId(),
>             _email,
>             _firstName,
>             _lastName,
>             _roles,
>             UserStatus.Active,
>             DateTime.UtcNow);
>     }
> 
>     private void ValidateRequiredFields()
>     {
>         if (string.IsNullOrEmpty(_email))
>             throw new ArgumentException("Email обязателен");
>         
>         if (_roles.Count == 0)
>             throw new ArgumentException("Пользователь должен иметь хотя бы одну роль");
>     }
> }
> 
> // Использование
> var user = new UserBuilder()
>     .WithEmail("john@example.com")
>     .WithName("John", "Doe")
>     .WithRole(Role.User)
>     .WithRole(Role.Editor)
>     .Build();
> ```

### 2. Abstract Factory для семейств объектов

> [!example] 🏭 **Абстрактная фабрика**
> 
> ```csharp
> public abstract class DocumentFactory
> {
>     public abstract IDocument CreateDocument();
>     public abstract IDocumentProcessor CreateProcessor();
>     public abstract IDocumentValidator CreateValidator();
> }
> 
> public class PdfDocumentFactory : DocumentFactory
> {
>     public override IDocument CreateDocument() => new PdfDocument();
>     public override IDocumentProcessor CreateProcessor() => new PdfProcessor();
>     public override IDocumentValidator CreateValidator() => new PdfValidator();
> }
> 
> public class WordDocumentFactory : DocumentFactory
> {
>     public override IDocument CreateDocument() => new WordDocument();
>     public override IDocumentProcessor CreateProcessor() => new WordProcessor();
>     public override IDocumentValidator CreateValidator() => new WordValidator();
> }
> ```

---

## Когда использовать Factory?

> [!question] 🤔 **Критерии использования**
> 
> **Используйте Factory когда:**
> 
> - ✅ Создание объекта требует сложной логики
> - ✅ Нужна валидация при создании
> - ✅ Требуются внешние сервисы для создания
> - ✅ Есть несколько способов создания объекта
> 
> **НЕ используйте Factory когда:**
> 
> - ❌ Объект создается простым конструктором
> - ❌ Нет сложной логики создания
> - ❌ Создание происходит в одном месте

---

> [!summary] 📝 **Заключение**
> 
> **Domain Factory** - мощный паттерн для:
> 
> - 🎯 Централизации логики создания объектов
> - ✅ Обеспечения корректности доменных объектов
> - 🔧 Скрытия сложности конструирования
> - 🧪 Улучшения тестируемости кода
> 
> Выбирайте подходящий тип Factory в зависимости от сложности ваших доменных объектов!

---
