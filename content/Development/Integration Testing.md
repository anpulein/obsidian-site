---
title: Integration Testing
aliases:
  - Integration Testing
linter-yaml-title-alias: Integration Testing
date created: Thursday, August 28th 2025, 10:50:57 am
date modified: Thursday, August 28th 2025, 10:53:15 am
tags:
  - type/area
  - area/architecture
  - concept/microservice
  - concept/clean-architecture
---
## 🏷️ Tags

#type/area #area/architecture #concept/microservice #concept/clean-architecture 

---

> [!info] Определение **Integration Testing** — это тестирование взаимодействия между различными компонентами приложения, включая базы данных, внешние сервисы, файловую систему и другие зависимости.

## 📋 Содержание

- [[#🎯 Основные концепции]]
- [[#🔧 Настройка тестовой среды]]
- [[#🏗️ TestServer и WebApplicationFactory]]
- [[#💾 Тестирование с базой данных]]
- [[#🌐 Моки внешних сервисов]]
- [[#📊 Лучшие практики]]

---

## 🎯 Основные концепции

### Отличие от Unit тестов

|Аспект|Unit Tests|Integration Tests|
|---|---|---|
|**Область**|Отдельные методы/классы|Компоненты и их взаимодействие|
|**Зависимости**|Мокируются|Реальные или близкие к реальным|
|**Скорость**|Быстрые|Медленнее|
|**Сложность настройки**|Минимальная|Высокая|

> [!tip] Когда использовать
> 
> - Тестирование API endpoints
> - Проверка работы с базой данных
> - Валидация бизнес-процессов
> - Тестирование middleware pipeline

---

## 🔧 Настройка тестовой среды

### Необходимые пакеты

```bash
dotnet add package Microsoft.AspNetCore.Mvc.Testing
dotnet add package Microsoft.EntityFrameworkCore.InMemory
dotnet add package Testcontainers
dotnet add package FluentAssertions
```

### Базовый класс тестов

```csharp
public class IntegrationTestBase : IClassFixture<WebApplicationFactory<Program>>
{
    protected readonly WebApplicationFactory<Program> _factory;
    protected readonly HttpClient _client;

    public IntegrationTestBase(WebApplicationFactory<Program> factory)
    {
        _factory = factory;
        _client = factory.CreateClient();
    }
}
```

---

## 🏗️ TestServer и WebApplicationFactory

### Настройка WebApplicationFactory

```csharp
public class CustomWebApplicationFactory<TStartup> : WebApplicationFactory<TStartup> 
    where TStartup : class
{
    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        builder.ConfigureServices(services =>
        {
            // Удаление production БД
            var descriptor = services.SingleOrDefault(d => 
                d.ServiceType == typeof(DbContextOptions<ApplicationDbContext>));
            
            if (descriptor != null)
                services.Remove(descriptor);

            // Добавление InMemory БД для тестов
            services.AddDbContext<ApplicationDbContext>(options =>
            {
                options.UseInMemoryDatabase("InMemoryDbForTesting");
            });
        });

        builder.UseEnvironment("Testing");
    }
}
```

### Пример теста API Controller

```csharp
public class ProductsControllerTests : IntegrationTestBase
{
    public ProductsControllerTests(WebApplicationFactory<Program> factory) : base(factory) { }

    [Fact]
    public async Task GetProducts_ReturnsSuccessStatusCode()
    {
        // Act
        var response = await _client.GetAsync("/api/products");

        // Assert
        response.EnsureSuccessStatusCode();
        
        var content = await response.Content.ReadAsStringAsync();
        var products = JsonSerializer.Deserialize<List<Product>>(content);
        
        products.Should().NotBeNull();
        products.Should().HaveCountGreaterThan(0);
    }

    [Fact]
    public async Task CreateProduct_ValidProduct_ReturnsCreated()
    {
        // Arrange
        var newProduct = new CreateProductRequest
        {
            Name = "Test Product",
            Price = 99.99m,
            CategoryId = 1
        };

        var json = JsonSerializer.Serialize(newProduct);
        var content = new StringContent(json, Encoding.UTF8, "application/json");

        // Act
        var response = await _client.PostAsync("/api/products", content);

        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.Created);
        
        var responseContent = await response.Content.ReadAsStringAsync();
        var createdProduct = JsonSerializer.Deserialize<Product>(responseContent);
        
        createdProduct.Name.Should().Be(newProduct.Name);
        createdProduct.Price.Should().Be(newProduct.Price);
    }
}
```

---

## 💾 Тестирование с базой данных

### In-Memory Database

> [!warning] Ограничения InMemory
> 
> - Не поддерживает все SQL функции
> - Отсутствуют referential constraints
> - Не тестирует реальную БД логику

```csharp
services.AddDbContext<AppDbContext>(options =>
    options.UseInMemoryDatabase(databaseName: Guid.NewGuid().ToString()));
```

### SQLite для тестов

```csharp
services.AddDbContext<AppDbContext>(options =>
    options.UseSqlite($"Data Source={Path.GetTempFileName()}"));
```

### Testcontainers (Рекомендуемый подход)

```csharp
public class DatabaseIntegrationTests : IAsyncLifetime
{
    private readonly PostgreSqlContainer _container = new PostgreSqlBuilder()
        .WithDatabase("testdb")
        .WithUsername("testuser")
        .WithPassword("testpass")
        .Build();

    public async Task InitializeAsync()
    {
        await _container.StartAsync();
    }

    public async Task DisposeAsync()
    {
        await _container.StopAsync();
    }

    [Fact]
    public async Task DatabaseOperations_WorkCorrectly()
    {
        // Arrange
        var connectionString = _container.GetConnectionString();
        
        var services = new ServiceCollection();
        services.AddDbContext<AppDbContext>(options =>
            options.UseNpgsql(connectionString));
        
        var serviceProvider = services.BuildServiceProvider();
        
        using var context = serviceProvider.GetRequiredService<AppDbContext>();
        await context.Database.EnsureCreatedAsync();

        // Act & Assert
        var product = new Product { Name = "Test", Price = 10.0m };
        context.Products.Add(product);
        await context.SaveChangesAsync();

        var savedProduct = await context.Products.FirstOrDefaultAsync();
        savedProduct.Should().NotBeNull();
        savedProduct.Name.Should().Be("Test");
    }
}
```

---

## 🌐 Моки внешних сервисов

### Мокирование HTTP клиентов

```csharp
public class ExternalServiceTests : IntegrationTestBase
{
    [Fact]
    public async Task CallExternalService_ReturnsExpectedData()
    {
        // Arrange
        var factory = _factory.WithWebHostBuilder(builder =>
        {
            builder.ConfigureTestServices(services =>
            {
                // Замена HttpClient для внешнего сервиса
                services.AddHttpClient<IExternalService, ExternalService>(client =>
                {
                    client.BaseAddress = new Uri("https://fake-api.com");
                })
                .AddHttpMessageHandler<FakeHttpMessageHandler>();
            });
        });

        var client = factory.CreateClient();

        // Act
        var response = await client.GetAsync("/api/external-data");

        // Assert
        response.EnsureSuccessStatusCode();
        var content = await response.Content.ReadAsStringAsync();
        content.Should().Contain("mocked data");
    }
}

public class FakeHttpMessageHandler : HttpMessageHandler
{
    protected override Task<HttpResponseMessage> SendAsync(
        HttpRequestMessage request, 
        CancellationToken cancellationToken)
    {
        var response = new HttpResponseMessage(HttpStatusCode.OK)
        {
            Content = new StringContent("{\"data\": \"mocked data\"}")
        };
        
        return Task.FromResult(response);
    }
}
```

### Переопределение сервисов

```csharp
builder.ConfigureTestServices(services =>
{
    services.Remove<IEmailService>();
    services.AddSingleton<IEmailService, FakeEmailService>();
});

public class FakeEmailService : IEmailService
{
    public List<Email> SentEmails { get; } = new();

    public Task SendEmailAsync(string to, string subject, string body)
    {
        SentEmails.Add(new Email(to, subject, body));
        return Task.CompletedTask;
    }
}
```

---

## 📊 Лучшие практики

### ✅ DO

> [!success] Рекомендации
> 
> - **Используйте реальные БД** для критичных тестов (Testcontainers)
> - **Изолируйте тесты** друг от друга
> - **Тестируйте end-to-end сценарии**
> - **Очищайте данные** после каждого теста
> - **Группируйте тесты** по функциональности

### ❌ DON'T

> [!error] Избегайте
> 
> - Совместного использования состояния между тестами
> - Тестирования implementation details
> - Слишком большого количества integration тестов
> - Игнорирования производительности тестов

### 🔄 Шаблон Arrange-Act-Assert

```csharp
[Fact]
public async Task UpdateProduct_ValidId_ReturnsUpdatedProduct()
{
    // Arrange - Подготовка данных
    using var scope = _factory.Services.CreateScope();
    var context = scope.ServiceProvider.GetRequiredService<AppDbContext>();
    
    var product = new Product { Name = "Original", Price = 50 };
    context.Products.Add(product);
    await context.SaveChangesAsync();

    var updateRequest = new UpdateProductRequest 
    { 
        Name = "Updated", 
        Price = 75 
    };

    // Act - Выполнение действия
    var response = await _client.PutAsJsonAsync($"/api/products/{product.Id}", updateRequest);

    // Assert - Проверка результата
    response.EnsureSuccessStatusCode();
    
    var updatedProduct = await response.Content.ReadFromJsonAsync<Product>();
    updatedProduct.Name.Should().Be("Updated");
    updatedProduct.Price.Should().Be(75);
    
    // Проверка в БД
    var dbProduct = await context.Products.FindAsync(product.Id);
    dbProduct.Name.Should().Be("Updated");
}
```

---

## 📚 Дополнительные ресурсы

> [!note] Полезные ссылки
> 
> - [Microsoft Docs: Integration tests](https://docs.microsoft.com/aspnet/core/test/integration-tests)
> - [Testcontainers Documentation](https://dotnet.testcontainers.org/)
> - [FluentAssertions Documentation](https://fluentassertions.com/)
