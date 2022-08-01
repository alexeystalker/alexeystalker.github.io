---
share: true
tags: [NET/async,NET/TaskFactory]
---
# Task.Factory.StartNew
Рассмотрим код
```csharp
class Program
{
    public static async Task ProcessAsync()
    {
        await Task.Delay(2000);
        Console.WriteLine("Processing done");
    }
    
    static async Task Main(string[] args)
    {
        await Task.Factory.StartNew(ProcessAsync);
        Console.WriteLine("End of program");
        Console.ReadLine();
    }
}
```
В этом коде `End of program` будет напечатано раньше, чем `Processing done`, потому что `Task.Factory.StartNew` вернет `Task<Task>`(Таск с результатом Таск!)  а `await` ждёт только "внешний" таск. Правильно было бы написать либо `Task.Factory.StartNew(ProcessAsync).Unwrap()`, либо `await Task.Run(ProcessAsync)`.

Существует три легитимных случая использования `Task.Factory.StartNew`
1. Запуск таска на планировщике тасков (TaskScheduler), отлично от дефолтного
2. Запуская таск на выделенном потоке (`TaskCreationOptions.LongRunning`)
3. Ставя таск в глобальную очередь тредпула (`TaskCreationOptions.PreferFairness`)

В остальных случаях лучше не использовать `Task.Factory.StartNew`.
## Ссылки
https://kevingosse.medium.com/performance-best-practices-in-c-b85a47bdd93a
