---
tags:
  - type/area
  - area/data
  - tech/csharp
  - "concept/perfomance"
  - "impl/yield"
  - "task/optimize"
title: Полное руководство по yield в C#
aliases:
  - Полное руководство по yield в C#
linter-yaml-title-alias: Полное руководство по yield в C#
date created: Thursday, August 7th 2025, 1:08:27 pm
date modified: Wednesday, August 13th 2025, 11:03:19 am
---
# Полное руководство по yield в C#

> [!info] Введение `yield` - это контекстное ключевое слово в C#, которое используется для создания итераторов. Оно позволяет возвращать элементы коллекции по одному, не создавая всю коллекцию в памяти сразу.

## 📋 Содержание

- [[#Что такое yield]]
- [[#Основы работы с yield]]
- [[#yield return vs yield break]]
- [[#Практические примеры]]
- [[#Продвинутые сценарии]]
- [[#Производительность и память]]
- [[#Ограничения и особенности]]
- [[#Лучшие практики]]

---

## Что такое yield

> [!note] Определение `yield` создает **ленивый итератор** - объект, который генерирует элементы по требованию, а не все сразу.

### Преимущества yield:

- ✅ **Экономия памяти** - элементы создаются по мере необходимости
- ✅ **Ленивые вычисления** - обработка только нужных элементов
- ✅ **Простота кода** - нет необходимости создавать классы итераторов
- ✅ **Композиция** - легко комбинировать итераторы

---

## Основы работы с yield

### Базовый синтаксис

```csharp
public static IEnumerable<int> GetNumbers()
{
    yield return 1;
    yield return 2;
    yield return 3;
}
```

> [!example] Простой пример
> 
> ```csharp
> static void Main()
> {
>     foreach (int number in GetNumbers())
>     {
>         Console.WriteLine(number); // Выведет: 1, 2, 3
>     }
> }
> ```

### Как это работает под капотом

```mermaid
graph TD
    A[Вызов GetNumbers()] --> B[Создается state machine]
    B --> C[foreach начинает итерацию]
    C --> D[Выполнение до первого yield return]
    D --> E[Возврат значения и приостановка]
    E --> F[Продолжение с места остановки]
    F --> G{Есть еще yield?}
    G -->|Да| D
    G -->|Нет| H[Завершение итерации]
```

---

## yield return vs yield break

### yield return

Возвращает элемент и приостанавливает выполнение метода.

```csharp
public static IEnumerable<string> GetNames()
{
    yield return "Alice";
    yield return "Bob";
    yield return "Charlie";
}
```

### yield break

Прерывает выполнение итератора (аналог `return` в обычном методе).

```csharp
public static IEnumerable<int> GetPositiveNumbers(int[] numbers)
{
    foreach (int number in numbers)
    {
        if (number < 0)
            yield break; // Прерываем итерацию при первом отрицательном числе
        
        yield return number;
    }
}
```

> [!warning] Важно! В методах с `yield` нельзя использовать обычный `return` для возврата значений.

---

## Практические примеры

### 1. Генерация последовательности Фибоначчи

```csharp
public static IEnumerable<long> Fibonacci()
{
    long a = 0, b = 1;
    
    while (true) // Бесконечная последовательность!
    {
        yield return a;
        (a, b) = (b, a + b);
    }
}

// Использование
var first10Fibonacci = Fibonacci().Take(10);
foreach (var fib in first10Fibonacci)
{
    Console.WriteLine(fib);
}
```

### 2. Чтение файла построчно

```csharp
public static IEnumerable<string> ReadLinesLazy(string filePath)
{
    using var reader = new StreamReader(filePath);
    string line;
    
    while ((line = reader.ReadLine()) != null)
    {
        yield return line;
    }
}

// Эффективно для больших файлов
foreach (string line in ReadLinesLazy("large-file.txt"))
{
    if (line.Contains("error"))
    {
        Console.WriteLine(line);
        break; // Остальные строки не будут прочитаны
    }
}
```

### 3. Фильтрация и преобразование

```csharp
public static IEnumerable<T> Where<T>(this IEnumerable<T> source, Func<T, bool> predicate)
{
    foreach (T item in source)
    {
        if (predicate(item))
            yield return item;
    }
}

public static IEnumerable<TResult> Select<T, TResult>(
    this IEnumerable<T> source, 
    Func<T, TResult> selector)
{
    foreach (T item in source)
    {
        yield return selector(item);
    }
}
```

---

## Продвинутые сценарии

### 1. Рекурсивный обход дерева

```csharp
public class TreeNode<T>
{
    public T Value { get; set; }
    public List<TreeNode<T>> Children { get; set; } = new();
    
    public IEnumerable<T> TraverseDepthFirst()
    {
        yield return Value;
        
        foreach (var child in Children)
        {
            foreach (var descendant in child.TraverseDepthFirst())
            {
                yield return descendant;
            }
        }
    }
}
```

### 2. Пакетная обработка

```csharp
public static IEnumerable<IEnumerable<T>> Batch<T>(
    this IEnumerable<T> source, 
    int batchSize)
{
    var batch = new List<T>(batchSize);
    
    foreach (T item in source)
    {
        batch.Add(item);
        
        if (batch.Count == batchSize)
        {
            yield return batch;
            batch = new List<T>(batchSize);
        }
    }
    
    if (batch.Count > 0)
        yield return batch;
}

// Использование
var numbers = Enumerable.Range(1, 100);
foreach (var batch in numbers.Batch(10))
{
    Console.WriteLine($"Batch: [{string.Join(", ", batch)}]");
}
```

### 3. Асинхронный yield (IAsyncEnumerable)

```csharp
public static async IAsyncEnumerable<string> FetchDataAsync()
{
    for (int i = 1; i <= 10; i++)
    {
        // Имитация асинхронной операции
        await Task.Delay(100);
        yield return $"Data item {i}";
    }
}

// Использование
await foreach (string item in FetchDataAsync())
{
    Console.WriteLine(item);
}
```

---

## Производительность и память

### Сравнение подходов

|Подход|Память|Время создания|Ленивость|
|---|---|---|---|
|`List<T>`|O(n)|Сразу все|❌|
|`yield`|O(1)|По требованию|✅|
|`Array`|O(n)|Сразу все|❌|

### Бенчмарк пример

```csharp
// Обычный подход - создает все элементы сразу
public static List<int> GetSquaresEager(int count)
{
    var result = new List<int>();
    for (int i = 0; i < count; i++)
    {
        result.Add(i * i);
    }
    return result;
}

// yield подход - создает элементы по требованию  
public static IEnumerable<int> GetSquaresLazy(int count)
{
    for (int i = 0; i < count; i++)
    {
        yield return i * i;
    }
}
```

> [!tip] Результаты бенчмарка Для 1,000,000 элементов:
> 
> - **Eager**: ~4MB памяти, 15ms создание
> - **Lazy**: ~40B памяти, <1ms создание (до первого элемента)

---

## Ограничения и особенности

### ❌ Что НЕЛЬЗЯ делать с yield

```csharp
// ОШИБКА: yield в try-catch с catch
public static IEnumerable<int> BadExample1()
{
    try
    {
        yield return 1; // Компиляционная ошибка
    }
    catch
    {
        // ...
    }
}

// ОШИБКА: yield в unsafe блоке
public static unsafe IEnumerable<int> BadExample2()
{
    yield return 1; // Компиляционная ошибка
}

// ОШИБКА: ref/out параметры
public static IEnumerable<int> BadExample3(ref int value)
{
    yield return value; // Компиляционная ошибка
}
```

### ✅ Что МОЖНО делать

```csharp
// try-finally разрешен
public static IEnumerable<string> GoodExample1()
{
    try
    {
        yield return "Hello";
        yield return "World";
    }
    finally
    {
        Console.WriteLine("Cleanup");
    }
}

// using разрешен
public static IEnumerable<string> GoodExample2(string path)
{
    using var reader = File.OpenText(path);
    string line;
    while ((line = reader.ReadLine()) != null)
    {
        yield return line;
    }
}
```

---

## Лучшие практики

### 1. 🎯 Используйте yield для больших коллекций

```csharp
// ✅ Хорошо - для больших данных
public static IEnumerable<LogEntry> ParseLargeLogFile(string path)
{
    foreach (string line in File.ReadLines(path))
    {
        if (TryParseLogEntry(line, out LogEntry entry))
            yield return entry;
    }
}

// ❌ Плохо - для маленьких коллекций
public static IEnumerable<int> GetSmallNumbers()
{
    yield return 1;
    yield return 2;
    yield return 3;
}
// Лучше: return new[] { 1, 2, 3 };
```

### 2. 🔄 Проверяйте параметры заранее

```csharp
public static IEnumerable<T> SafeWhere<T>(
    this IEnumerable<T> source, 
    Func<T, bool> predicate)
{
    // Проверки выполняются сразу при вызове
    if (source == null) throw new ArgumentNullException(nameof(source));
    if (predicate == null) throw new ArgumentNullException(nameof(predicate));
    
    return WhereIterator(source, predicate);
}

private static IEnumerable<T> WhereIterator<T>(
    IEnumerable<T> source, 
    Func<T, bool> predicate)
{
    foreach (T item in source)
    {
        if (predicate(item))
            yield return item;
    }
}
```

### 3. 📊 Документируйте поведение

```csharp
/// <summary>
/// Генерирует бесконечную последовательность простых чисел.
/// </summary>
/// <remarks>
/// ВНИМАНИЕ: Последовательность бесконечна! 
/// Используйте Take() или другие методы ограничения.
/// </remarks>
/// <returns>Ленивая последовательность простых чисел</returns>
public static IEnumerable<int> GeneratePrimes()
{
    yield return 2;
    
    var primes = new List<int> { 2 };
    int candidate = 3;
    
    while (true)
    {
        bool isPrime = true;
        foreach (int prime in primes)
        {
            if (prime * prime > candidate) break;
            if (candidate % prime == 0)
            {
                isPrime = false;
                break;
            }
        }
        
        if (isPrime)
        {
            primes.Add(candidate);
            yield return candidate;
        }
        
        candidate += 2;
    }
}
```

---

## Заключение

> [!success] Ключевые выводы
> 
> - `yield` идеален для работы с большими объемами данных
> - Обеспечивает ленивые вычисления и экономию памяти
> - Упрощает создание итераторов
> - Имеет ограничения, которые нужно учитывать
> - Требует осторожности с бесконечными последовательностями

> [!quote] Помните _"yield - это не просто синтаксический сахар, это мощный инструмент для создания эффективных и элегантных решений в C#"_

---

## Дополнительные ресурсы

- 📖 [Microsoft Docs: yield](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/yield)
- 🎥 [Eric Lippert: How yield works](https://blogs.msdn.microsoft.com/ericlippert/)
- 📚 [C# in Depth by Jon Skeet](https://www.manning.com/books/c-sharp-in-depth-fourth-edition)
