---
title: ArchUnitNet
aliases:
  - ArchUnitNet
linter-yaml-title-alias: ArchUnitNet
date created: Thursday, August 28th 2025, 10:57:19 am
date modified: Friday, August 29th 2025, 7:26:12 am
tags:
  - type/area
  - area/architecture
  - concept/microservice
  - concept/clean-architecture
---
## 🏷️ Tags

#type/area #area/architecture #concept/microservice #concept/clean-architecture 

---

> [!info] Что это такое? **ArchUnitNet** — это библиотека для архитектурного тестирования в .NET, которая позволяет писать автоматизированные тесты для проверки соблюдения архитектурных правил в коде.

## 🎯 Основные возможности

- ✅ Проверка зависимостей между слоями
- ✅ Контроль именования классов и методов
- ✅ Валидация структуры проекта
- ✅ Проверка соблюдения принципов SOLID
- ✅ Интеграция с популярными тестовыми фреймворками

---

## 📦 Установка

```bash
dotnet add package TngTech.ArchUnitNET.xUnit
```

> [!tip] Альтернативы
> 
> - `TngTech.ArchUnitNET.NUnit` для NUnit
> - `TngTech.ArchUnitNET.MSTestV2` для MSTest

---

## 🏗️ Базовая структура теста

```csharp
using ArchUnitNET.Domain;
using ArchUnitNET.Loader;
using ArchUnitNET.Fluent;
using Xunit;

public class ArchitectureTests
{
    private static readonly Architecture Architecture = 
        new ArchLoader().LoadAssemblies(
            System.Reflection.Assembly.GetExecutingAssembly()
        ).Build();

    [Fact]
    public void Controllers_Should_Have_Controller_Suffix()
    {
        var rule = ArchRuleDefinition.Classes()
            .That().ResideInNamespace("MyApp.Controllers")
            .Should().HaveNameEndingWith("Controller");

        rule.Check(Architecture);
    }
}
```

---

## 🔄 Проверка зависимостей между слоями

### Классический трехслойный подход

```csharp
[Fact]
public void Domain_Should_Not_Depend_On_Infrastructure()
{
    var rule = ArchRuleDefinition.Classes()
        .That().ResideInNamespace("MyApp.Domain")
        .Should().NotDependOnAny("MyApp.Infrastructure");

    rule.Check(Architecture);
}

[Fact] 
public void Controllers_Should_Not_Depend_On_Infrastructure_Directly()
{
    var rule = ArchRuleDefinition.Classes()
        .That().ResideInNamespace("MyApp.Controllers")
        .Should().NotDependOnAny("MyApp.Infrastructure")
        .Because("Controllers should depend only on Application layer");

    rule.Check(Architecture);
}
```

### Clean Architecture

```csharp
[Fact]
public void Core_Should_Not_Have_External_Dependencies()
{
    var coreTypes = ArchRuleDefinition.Classes()
        .That().ResideInNamespace("MyApp.Core");

    var rule = coreTypes.Should()
        .NotDependOnAny("MyApp.Infrastructure")
        .AndShould().NotDependOnAny("MyApp.Web")
        .AndShould().NotDependOnAny("System.Data");

    rule.Check(Architecture);
}
```

---

## 📝 Именование и структура

### Контроллеры

```csharp
[Fact]
public void Controllers_Should_Have_Proper_Structure()
{
    // Именование
    var namingRule = ArchRuleDefinition.Classes()
        .That().ResideInNamespace("MyApp.Controllers")
        .Should().HaveNameEndingWith("Controller");

    // Наследование
    var inheritanceRule = ArchRuleDefinition.Classes()
        .That().ResideInNamespace("MyApp.Controllers") 
        .Should().BeAssignableTo("Microsoft.AspNetCore.Mvc.Controller");

    // Атрибуты
    var attributeRule = ArchRuleDefinition.Classes()
        .That().ResideInNamespace("MyApp.Controllers")
        .Should().BeDecoratedWith("Microsoft.AspNetCore.Mvc.ApiControllerAttribute");

    namingRule.Check(Architecture);
    inheritanceRule.Check(Architecture);
    attributeRule.Check(Architecture);
}
```

### Сервисы

```csharp
[Fact]
public void Services_Should_Have_Interface()
{
    var rule = ArchRuleDefinition.Classes()
        .That().ResideInNamespace("MyApp.Services")
        .And().HaveNameEndingWith("Service")
        .Should().ImplementInterface("I.*Service");

    rule.Check(Architecture);
}

[Fact]
public void Service_Interfaces_Should_Be_In_Contracts()
{
    var rule = ArchRuleDefinition.Interfaces()
        .That().HaveNameStartingWith("I")
        .And().HaveNameEndingWith("Service")
        .Should().ResideInNamespace("MyApp.Contracts");

    rule.Check(Architecture);
}
```

---

## 🔒 Проверка инкапсуляции

```csharp
[Fact]
public void Domain_Entities_Should_Not_Have_Public_Setters()
{
    var rule = ArchRuleDefinition.Classes()
        .That().ResideInNamespace("MyApp.Domain.Entities")
        .Should().NotHaveAnyPublicPropertySetters();

    rule.Check(Architecture);
}

[Fact]
public void Private_Fields_Should_Have_Underscore_Prefix()
{
    var rule = ArchRuleDefinition.Fields()
        .That().ArePrivate()
        .Should().HaveNameStartingWith("_");

    rule.Check(Architecture);
}
```

---

## 🎭 Продвинутые паттерны

### Repository Pattern

```csharp
[Fact]
public void Repositories_Should_Follow_Pattern()
{
    // Интерфейсы репозиториев
    var interfaceRule = ArchRuleDefinition.Interfaces()
        .That().HaveNameEndingWith("Repository")
        .Should().ResideInNamespace("MyApp.Domain.Repositories");

    // Реализации репозиториев  
    var implementationRule = ArchRuleDefinition.Classes()
        .That().HaveNameEndingWith("Repository")
        .And().DoNotHaveNameStartingWith("I")
        .Should().ResideInNamespace("MyApp.Infrastructure.Repositories")
        .AndShould().ImplementInterface(".*Repository");

    interfaceRule.Check(Architecture);
    implementationRule.Check(Architecture);
}
```

### CQRS Pattern

```csharp
[Fact] 
public void CQRS_Commands_And_Queries_Should_Be_Separated()
{
    // Команды
    var commandRule = ArchRuleDefinition.Classes()
        .That().HaveNameEndingWith("Command")
        .Should().ResideInNamespace("MyApp.Application.Commands")
        .AndShould().BeSealed();

    // Запросы
    var queryRule = ArchRuleDefinition.Classes()
        .That().HaveNameEndingWith("Query") 
        .Should().ResideInNamespace("MyApp.Application.Queries")
        .AndShould().BeSealed();

    // Обработчики
    var handlerRule = ArchRuleDefinition.Classes()
        .That().HaveNameEndingWith("Handler")
        .Should().ResideInNamespace("MyApp.Application.Handlers");

    commandRule.Check(Architecture);
    queryRule.Check(Architecture); 
    handlerRule.Check(Architecture);
}
```

---

## 🚫 Запрещенные зависимости

```csharp
[Fact]
public void Should_Not_Use_Concrete_Logging()
{
    var rule = ArchRuleDefinition.Classes()
        .Should().NotDependOnAny("NLog", "Serilog", "log4net")
        .Because("Should use Microsoft.Extensions.Logging abstractions");

    rule.Check(Architecture);
}

[Fact]
public void Domain_Should_Not_Use_Entity_Framework()
{
    var rule = ArchRuleDefinition.Classes()
        .That().ResideInNamespace("MyApp.Domain")
        .Should().NotDependOnAny("Microsoft.EntityFrameworkCore")
        .Because("Domain should be persistence ignorant");

    rule.Check(Architecture);
}
```

---

## 📊 Метрики и ограничения

```csharp
[Fact]
public void Classes_Should_Not_Be_Too_Large()
{
    var rule = ArchRuleDefinition.Classes()
        .That().AreNotInterfaces()
        .Should().NotHaveAnyMembersThat()
        .Are(MethodMembers()).And().HaveNameStartingWith("Method")
        .AndHaveCountGreaterThan(20)
        .Because("Classes should follow SRP");

    rule.Check(Architecture);
}

[Fact]
public void Cyclic_Dependencies_Should_Not_Exist()
{
    var rule = ArchRuleDefinition.Slices()
        .Matching("MyApp.(*)")
        .Should().BeFreeOfCycles();

    rule.Check(Architecture);
}
```

---

## 🔧 Настройки и конфигурация

### Исключения из правил

```csharp
[Fact]
public void Controllers_Except_Base_Should_Have_Suffix()
{
    var rule = ArchRuleDefinition.Classes()
        .That().ResideInNamespace("MyApp.Controllers")
        .And().DoNotHaveNameMatching(".*BaseController")
        .Should().HaveNameEndingWith("Controller");

    rule.Check(Architecture);
}
```

### Кастомные правила

```csharp
public static IArchRule CreateCustomRule()
{
    return ArchRuleDefinition.Classes()
        .That().ImplementInterface("IDisposable")
        .Should().HaveNameEndingWith("Disposable")
        .OrShould().BeSealed()
        .Because("Disposable resources should be properly managed");
}
```

---

## 📋 Рекомендации и best practices

> [!success] Лучшие практики
> 
> 1. **Запускайте тесты на CI/CD** - архитектурные правила должны проверяться автоматически
> 2. **Группируйте правила по контексту** - создавайте отдельные классы для разных аспектов архитектуры
> 3. **Используйте описательные сообщения** - добавляйте `.Because()` для объяснения правил
> 4. **Начинайте с простого** - внедряйте правила постепенно

> [!warning] Частые ошибки
> 
> - Слишком строгие правила на старте проекта
> - Отсутствие исключений для legacy кода
> - Игнорирование failed тестов
> - Дублирование правил в разных тестах

---

## 🔗 Полезные ссылки

- [Официальная документация](https://archunitnet.readthedocs.io/)
- [GitHub репозиторий](https://github.com/TNG/ArchUnitNET)
- [Примеры использования](https://github.com/TNG/ArchUnitNET/tree/main/ExampleTest)

---

