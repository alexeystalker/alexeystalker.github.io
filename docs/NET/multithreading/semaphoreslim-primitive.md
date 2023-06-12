---
share: true
aliases:
 - semaphoreslim
tags:
 - NET/multithreading
---
# SemaphoreSlim
`SemaphoreSlim` — примитив синхронизации, аналогичный Semaphore, но более легковесный, и не использующий примитивы операционной системы. Из-за этого он рекомендуется для синхронизации потоков в одном приложении.
При создании семафора указывается, сколько потоков могут одновременно зайти в критическую секцию. При этом существует конструктор с двумя параметрами — начальным и максимальным числом допустимых потоков. Вызов метода `Wait` или `WaitAsync` уменьшает счётчик допустимых потоков, а `Release` — увеличивает. При этом существует перегрузка `Release` с параметром, указывающим, насколько увеличить счётчик. Это можно использовать, к примеру, для того, чтобы сперва создать ожидающие таски, и только потом их запустить, вызвав `Release` с необходимым значением счётчика потоков.
## Пример использования
Пусть мы хотим выполнить несколько тасок над коллекцией однотипных данных. Первое, что приходит на ум:
```csharp
public async Task ProcessManyItems(List<string> items)
{
  var tasks = items.Select(
    async item => await ProcessItem(item));
  await Task.WhenAll(tasks);
}
```
Однако, в случае достаточно большого числа элементов будет создано слишком много потоков. Мы хотим ограничить параллелизм, это можно сделать (помимо других вариантов), используя `SemaphoreSlim` для контроля за числом потоков, одновременно входящих в критическую секцию:
```csharp
public async Task ProcessManyItems( List<string> items,  int maxConcurrency = 10)
{
  using (var semaphore = new SemaphoreSlim(maxConcurrency))
  {
    var tasks = items.Select(async item =>
     {
       // Таск завершается, когда есть свободное место
       await semaphore.WaitAsync(); 
       try
       {
         await ProcessItem(item);
       }
       finally
       {
         semaphore.Release();
       }
     });
     await Task.WhenAll(tasks);
  }
}
```

## Ссылки
[Документация](https://learn.microsoft.com/en-us/dotnet/api/system.threading.semaphoreslim?view=net-7.0)
[Статья с примером](https://dev.to/bytehide/50-c-advanced-optimization-performance-tips-18l2)