---
title: Architecture Tests
aliases:
  - Architecture Tests
linter-yaml-title-alias: Architecture Tests
date created: Thursday, August 28th 2025, 10:56:27 am
date modified: Thursday, August 28th 2025, 10:57:20 am
tags:
  - type/area
  - area/architecture
  - concept/microservice
  - concept/clean-architecture
---
## 🏷️ Tags

#type/area #area/architecture #concept/microservice #concept/clean-architecture 

---

> [!abstract] Что это такое? **Архитектурные тесты** — это автоматизированные тесты, которые проверяют соблюдение архитектурных правил и ограничений в кодовой базе. Они гарантируют, что код следует заданной архитектуре проекта.

---

## 🎯 Зачем нужны Architecture Tests?

> [!success] Преимущества
> 
> - ✅ **Соблюдение архитектуры** — автоматическая проверка правил
> - ✅ **Предотвращение деградации** — код не "портится" со временем
> - ✅ **Документирование архитектуры** — тесты как живая документация
> - ✅ **Раннее обнаружение** — проблемы находятся на этапе CI/CD

---

## 📚 Популярные библиотеки

|Библиотека|Описание|GitHub Stars|
|---|---|---|
|**NetArchTest**|Простая и мощная библиотека для .NET|⭐ 1.3k+|
|**ArchUnitNET**|Порт Java ArchUnit для .NET|⭐ 800+|
|**Mono.Cecil**|Низкоуровневая работа с IL|⭐ 2.7k+|

---

## 🛠️ NetArchTest - Основы

### Установка

```bash
dotnet add package NetArchTest.Rules
```

### Базовая структура теста

```csharp
[Test]
public void Controllers_Should_Have_Correct_Dependencies()
{
    var result = Types.InCurrentDomain()
        .That()
        .ResideInNamespace("MyApp.Controllers")
        .Should()
        .HaveDependencyOn("MyApp.Services")
        .GetResult();
    
    Assert.IsTrue(result.IsSuccessful);
}
```

---

## 🏗️ Практические примеры

### 1. Проверка зависимостей слоев

> [!example] Clean Architecture Контроллеры не должны напрямую обращаться к базе данных

```csharp
[Test]
public void Controllers_Should_Not_Access_Data_Layer_Directly()
{
    var result = Types.InCurrentDomain()
        .That()
        .ResideInNamespace("MyApp.Api.Controllers")
        .Should()
        .NotHaveDependencyOn("MyApp.Data")
        .GetResult();
        
    Assert.IsTrue(result.IsSuccessful, 
        string.Join(Environment.NewLine, result.FailingTypeNames));
}
```

### 2. Проверка именования

> [!tip] Конвенции именования Все контроллеры должны заканчиваться на "Controller"

```csharp
[Test]
public void Controllers_Should_Have_Controller_Suffix()
{
    var result = Types.InCurrentDomain()
        .That()
        .ResideInNamespace("MyApp.Controllers")
        .And()
        .Inherit(typeof(ControllerBase))
        .Should()
        .HaveNameEndingWith("Controller")
        .GetResult();
        
    Assert.IsTrue(result.IsSuccessful);
}
```

### 3. Проверка атрибутов

> [!warning] Важно для API Все публичные контроллеры должны иметь атрибут авторизации

```csharp
[Test]
public void Public_Controllers_Should_Have_Authorization()
{
    var result = Types.InCurrentDomain()
        .That()
        .ResideInNamespace("MyApp.Api.Controllers")
        .And()
        .ArePublic()
        .Should()
        .BeDecoratedWith(typeof(AuthorizeAttribute))
        .GetResult();
        
    Assert.IsTrue(result.IsSuccessful);
}
```

### 4. Проверка использования интерфейсов

> [!info] Принцип инверсии зависимостей Сервисы должны зависеть от абстракций, а не от конкретных реализаций

```csharp
[Test]
public void Services_Should_Depend_On_Interfaces_Only()
{
    var result = Types.InCurrentDomain()
        .That()
        .ResideInNamespace("MyApp.Services")
        .Should()
        .OnlyHaveDependenciesOn(
            "System",
            "Microsoft",
            "MyApp.Interfaces",
            "MyApp.Models"
        )
        .GetResult();
        
    Assert.IsTrue(result.IsSuccessful);
}
```

---

## 🎭 Сложные сценарии

### Кастомные правила с предикатами

```csharp
[Test]
public void Domain_Models_Should_Not_Reference_Infrastructure()
{
    var domainTypes = Types.InCurrentDomain()
        .That()
        .ResideInNamespace("MyApp.Domain");
        
    var infrastructureTypes = Types.InCurrentDomain()
        .That()
        .ResideInNamespace("MyApp.Infrastructure")
        .GetTypes();
        
    var result = domainTypes
        .Should()
        .NotHaveDependencyOnAny(infrastructureTypes.Select(t => t.FullName).ToArray())
        .GetResult();
        
    Assert.IsTrue(result.IsSuccessful);
}
```

### Проверка цикличных зависимостей

```csharp
[Test]
public void Check_For_Circular_Dependencies()
{
    var types = Types.InCurrentDomain()
        .That()
        .ResideInNamespace("MyApp")
        .GetTypes();
        
    // Кастомная логика для поиска циклических зависимостей
    var circularDeps = FindCircularDependencies(types);
    
    Assert.IsEmpty(circularDeps, 
        $"Found circular dependencies: {string.Join(", ", circularDeps)}");
}
```

---

## 🔧 Интеграция с CI/CD

### GitHub Actions

```yaml
name: Architecture Tests
on: [push, pull_request]

jobs:
  architecture-tests:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Setup .NET
      uses: actions/setup-dotnet@v3
      with:
        dotnet-version: '8.0.x'
    - name: Run Architecture Tests
      run: dotnet test --filter Category=Architecture
```

> [!note] Категоризация тестов Используйте атрибут `[Category("Architecture")]` для группировки архитектурных тестов

---

## 📊 Метрики и отчеты

### Пример отчета о нарушениях

```csharp
[Test]
public void Generate_Architecture_Report()
{
    var rules = new[]
    {
        ("Controllers naming", CheckControllerNaming()),
        ("Layer dependencies", CheckLayerDependencies()),
        ("Interface usage", CheckInterfaceUsage())
    };
    
    var report = new StringBuilder();
    report.AppendLine("# Architecture Test Report");
    report.AppendLine($"Generated: {DateTime.Now:yyyy-MM-dd HH:mm:ss}");
    report.AppendLine();
    
    foreach (var (name, result) in rules)
    {
        report.AppendLine($"## {name}");
        report.AppendLine(result.IsSuccessful ? "✅ PASSED" : "❌ FAILED");
        
        if (!result.IsSuccessful)
        {
            report.AppendLine("**Violations:**");
            foreach (var violation in result.FailingTypeNames)
            {
                report.AppendLine($"- {violation}");
            }
        }
        report.AppendLine();
    }
    
    File.WriteAllText("architecture-report.md", report.ToString());
}
```

---

## 🎯 Best Practices

> [!tip] Рекомендации
> 
> 1. **Начинайте с простого** — добавляйте правила постепенно
> 2. **Документируйте правила** — каждый тест должен объяснять "почему"
> 3. **Используйте осмысленные имена** тестов
> 4. **Группируйте по категориям** для удобства запуска
> 5. **Интегрируйте в CI/CD** на раннем этапе

> [!warning] Подводные камни
> 
> - ❌ Не создавайте слишком строгие правила сразу
> - ❌ Не забывайте обновлять тесты при изменении архитектуры
> - ❌ Не игнорируйте падающие архитектурные тесты

---

## 📖 Полезные ссылки

- 🔗 [NetArchTest GitHub](https://github.com/BenMorris/NetArchTest)
- 🔗 [ArchUnit for .NET](https://github.com/TNG/ArchUnitNET)
- 🔗 [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)

---

> [!quote] Помните _"Architecture is about the important stuff. Whatever that is."_ — Ralph Johnson
> 
> Архитектурные тесты помогают сохранить это "важное" в чистоте и порядке! 🏗️✨

---
