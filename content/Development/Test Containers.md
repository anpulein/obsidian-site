---
title: Test Containers
aliases:
  - Test Containers
linter-yaml-title-alias: Test Containers
date created: Thursday, August 28th 2025, 10:53:15 am
date modified: Thursday, August 28th 2025, 10:56:27 am
tags:
  - type/area
  - area/architecture
  - concept/microservice
  - concept/clean-architecture
  - concept/ddd
---
## 🏷️ Tags

#type/area #area/architecture #concept/microservice #concept/clean-architecture 

---

> [!info]+ Что такое TestContainers? TestContainers - это библиотека для интеграционного тестирования, которая позволяет запускать реальные сервисы (базы данных, очереди сообщений и др.) в Docker контейнерах во время выполнения тестов.

---

## 🎯 Основные преимущества

- **Изоляция** - каждый тест получает чистое окружение
- **Реалистичность** - тесты работают с реальными сервисами
- **Портативность** - одинаковое поведение на всех машинах
- **Автоматизация** - контейнеры создаются и удаляются автоматически

---

## 📦 Установка

```bash
dotnet add package Testcontainers
dotnet add package Testcontainers.PostgreSql
dotnet add package Testcontainers.Redis
# Другие специализированные пакеты по необходимости
```

---

## 🚀 Базовое использование

### Простой контейнер PostgreSQL

```csharp
using Testcontainers.PostgreSql;
using Xunit;

public class DatabaseTests : IAsyncLifetime
{
    private readonly PostgreSqlContainer _container;
    
    public DatabaseTests()
    {
        _container = new PostgreSqlBuilder()
            .WithDatabase("testdb")
            .WithUsername("testuser")
            .WithPassword("testpass")
            .Build();
    }
    
    public async Task InitializeAsync()
    {
        await _container.StartAsync();
    }
    
    public async Task DisposeAsync()
    {
        await _container.StopAsync();
        await _container.DisposeAsync();
    }
    
    [Fact]
    public async Task Should_Connect_To_Database()
    {
        // Arrange
        var connectionString = _container.GetConnectionString();
        
        // Act & Assert
        await using var connection = new NpgsqlConnection(connectionString);
        await connection.OpenAsync();
        Assert.Equal(ConnectionState.Open, connection.State);
    }
}
```

---

## 🔧 Продвинутые сценарии

### Настройка контейнера с кастомными параметрами

```csharp
var container = new PostgreSqlBuilder()
    .WithImage("postgres:15-alpine")
    .WithPortBinding(5432, true) // Случайный порт хоста
    .WithEnvironment("POSTGRES_INITDB_ARGS", "--auth-host=scram-sha-256")
    .WithVolumeMount("./init.sql", "/docker-entrypoint-initdb.d/init.sql")
    .WithWaitStrategy(Wait.ForUnixContainer()
        .UntilPortIsAvailable(5432)
        .UntilCommandIsCompleted("pg_isready"))
    .WithCleanUp(true)
    .Build();
```

### Использование с Docker Compose

```csharp
public class IntegrationTests : IAsyncLifetime
{
    private readonly DockerComposeEnvironment _environment;
    
    public IntegrationTests()
    {
        _environment = new DockerComposeBuilder()
            .WithDockerComposeFile("docker-compose.test.yml")
            .WithEnvironment("TEST_ENV", "true")
            .Build();
    }
    
    public async Task InitializeAsync()
    {
        await _environment.UpAsync();
    }
    
    public async Task DisposeAsync()
    {
        await _environment.DownAsync();
    }
}
```

---

## 🏗️ Паттерны использования

### Shared Container Pattern

> [!tip]+ Когда использовать Используйте для медленно стартующих контейнеров, которые можно переиспользовать между тестами

```csharp
public class SharedContainerFixture : IAsyncLifetime
{
    public PostgreSqlContainer Container { get; private set; }
    
    public async Task InitializeAsync()
    {
        Container = new PostgreSqlBuilder().Build();
        await Container.StartAsync();
    }
    
    public async Task DisposeAsync()
    {
        await Container?.DisposeAsync();
    }
}

public class DatabaseTests : IClassFixture<SharedContainerFixture>
{
    private readonly SharedContainerFixture _fixture;
    
    public DatabaseTests(SharedContainerFixture fixture)
    {
        _fixture = fixture;
    }
    
    [Fact]
    public async Task Test_With_Shared_Container()
    {
        var connectionString = _fixture.Container.GetConnectionString();
        // Тест логика...
    }
}
```

### Factory Pattern для создания контейнеров

```csharp
public static class TestContainerFactory
{
    public static PostgreSqlContainer CreatePostgreSql() =>
        new PostgreSqlBuilder()
            .WithImage("postgres:15-alpine")
            .WithDatabase("testdb")
            .WithUsername("test")
            .WithPassword("test")
            .Build();
    
    public static RedisContainer CreateRedis() =>
        new RedisBuilder()
            .WithImage("redis:7-alpine")
            .Build();
}
```

---

## 🧪 Тестирование с реальными сценариями

### Тестирование Repository с EF Core

```csharp
public class UserRepositoryTests : IAsyncLifetime
{
    private readonly PostgreSqlContainer _container;
    private DbContext _context;
    
    public UserRepositoryTests()
    {
        _container = TestContainerFactory.CreatePostgreSql();
    }
    
    public async Task InitializeAsync()
    {
        await _container.StartAsync();
        
        var options = new DbContextOptionsBuilder<AppDbContext>()
            .UseNpgsql(_container.GetConnectionString())
            .Options;
            
        _context = new AppDbContext(options);
        await _context.Database.EnsureCreatedAsync();
    }
    
    [Fact]
    public async Task Should_Save_And_Retrieve_User()
    {
        // Arrange
        var repository = new UserRepository(_context);
        var user = new User { Name = "John Doe", Email = "john@example.com" };
        
        // Act
        await repository.AddAsync(user);
        await _context.SaveChangesAsync();
        
        // Assert
        var savedUser = await repository.GetByIdAsync(user.Id);
        Assert.NotNull(savedUser);
        Assert.Equal("John Doe", savedUser.Name);
    }
    
    public async Task DisposeAsync()
    {
        await _context?.DisposeAsync();
        await _container?.DisposeAsync();
    }
}
```

### Тестирование API контроллеров

```csharp
public class ApiIntegrationTests : IAsyncLifetime
{
    private readonly PostgreSqlContainer _dbContainer;
    private readonly RedisContainer _cacheContainer;
    private WebApplication _app;
    private HttpClient _client;
    
    public async Task InitializeAsync()
    {
        _dbContainer = TestContainerFactory.CreatePostgreSql();
        _cacheContainer = TestContainerFactory.CreateRedis();
        
        await Task.WhenAll(
            _dbContainer.StartAsync(),
            _cacheContainer.StartAsync()
        );
        
        var builder = WebApplication.CreateBuilder();
        
        // Настройка сервисов с тестовыми контейнерами
        builder.Services.AddDbContext<AppDbContext>(options =>
            options.UseNpgsql(_dbContainer.GetConnectionString()));
            
        builder.Services.AddStackExchangeRedisCache(options =>
            options.Configuration = _cacheContainer.GetConnectionString());
        
        _app = builder.Build();
        _client = _app.GetTestClient();
    }
    
    [Fact]
    public async Task Should_Create_User_Via_Api()
    {
        // Arrange
        var createUserRequest = new { Name = "Jane Doe", Email = "jane@example.com" };
        
        // Act
        var response = await _client.PostAsJsonAsync("/api/users", createUserRequest);
        
        // Assert
        response.EnsureSuccessStatusCode();
        var user = await response.Content.ReadFromJsonAsync<User>();
        Assert.NotNull(user);
        Assert.Equal("Jane Doe", user.Name);
    }
}
```

---

## 📊 Мониторинг и отладка

### Логирование контейнеров

```csharp
var container = new PostgreSqlBuilder()
    .WithLogger(LoggerFactory.Create(builder => builder.AddConsole()).CreateLogger("TestContainer"))
    .Build();

// Получение логов контейнера
await container.StartAsync();
var logs = await container.GetLogsAsync();
Console.WriteLine(logs.Stdout);
```

### Выполнение команд в контейнере

```csharp
// Выполнение команды в работающем контейнере
var result = await container.ExecAsync(new[] { "psql", "-U", "postgres", "-c", "SELECT version();" });
Console.WriteLine($"Exit code: {result.ExitCode}");
Console.WriteLine($"Output: {result.Stdout}");
```

---

## ⚡ Best Practices

> [!warning]+ Производительность
> 
> - Используйте легковесные образы (alpine variants)
> - Переиспользуйте контейнеры между тестами когда возможно
> - Настраивайте таймауты ожидания разумно

> [!success]+ Надежность
> 
> - Всегда реализуйте `IAsyncLifetime` или используйте `using`
> - Настраивайте правильные стратегии ожидания (Wait Strategies)
> - Используйте случайные порты для избежания конфликтов

```csharp
// ✅ Хорошо - правильная очистка ресурсов
await using var container = new PostgreSqlBuilder().Build();
await container.StartAsync();

// ✅ Хорошо - кастомная стратегия ожидания
var container = new PostgreSqlBuilder()
    .WithWaitStrategy(Wait.ForUnixContainer()
        .UntilPortIsAvailable(5432)
        .UntilHttpRequestIsSucceeded(r => r.ForPort(8080).ForPath("/health"))
        .UntilCommandIsCompleted("pg_isready", "-h", "localhost"))
    .Build();
```

---

## 🔗 Полезные ресурсы

- [[Docker Commands]] - базовые команды Docker
- [[Integration Testing Patterns]] - паттерны интеграционного тестирования
- [[EF Core Testing]] - тестирование с Entity Framework Core

---

## 📝 Заключение

TestContainers кардинально улучшает качество интеграционных тестов, предоставляя реальное окружение без сложной настройки. Основные принципы:

1. **Изолируйте** - каждый тест должен иметь чистое состояние
2. **Автоматизируйте** - контейнеры управляются программно
3. **Оптимизируйте** - используйте shared containers для тяжелых сервисов
4. **Очищайте** - всегда корректно освобождайте ресурсы

> [!quote]+ 💡 Помните "Лучший тест - это тест, который максимально близок к продакшену" - TestContainers делает это возможным!

---
