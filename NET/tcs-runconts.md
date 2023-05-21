---
share: true
tags: [NET/async,NET/TaskCompletionSource,NET/multithreading]
---
#  TaskCompletionSource и продолжения в том же потоке.
Рассмотрим код
```csharp
class Program
{
    private static ManualResetEventSlim _mutex = new ManualResetEventSlim();
    
    public static async Task Deadlock()
    {
        await ProcessAsync();
        _mutex.Wait();
    }
    
    private static Task ProcessAsync()
    {
        var tcs = new TaskCompletionSource<bool>();
        
        Task.Run(() =>
        {
            Thread.Sleep(2000); // Simulate some work
            tcs.SetResult(true);
            _mutex.Set();
        });
        
        return tcs.Task;
    }
    
    static void Main(string[] args)
    {
        Deadlock().Wait();
        Console.WriteLine("Will never get there");
    }
}
```
Продолжение выполнения при использовании [[task-completion-source|TaskCompletionSource]] обычно происходит в текущем потоке. В этом коде, после выполнения `tcs.SetResult()` ожидание `ProcessAsync()` завершится, и таким образом, [[manual-reset-event-slim|мьютекс]] (`_mutex.Wait()`) будет выполняться в том же потоке, который должен вызвать `_mutex.Set()`,  что приведёт к дэдлоку. Чтобы этого избежать, нужно передавать настройку `TaskCreationsOptions.RunContinuationsAsyncronously` при создании `TaskCompletionSource`.
**Нужно всегда хорошо подумать, прежде чем НЕ использовать эту настройку!**
> [!warning] Внимание!
> Также важно не перепутать эту настройку с `TaskContinuationOptions.RunContinuationsAsyncronously` - код скомпилируется, но настройка будет проигнорирована и выполнение продолжится в том же потоке.
## Ссылки
https://kevingosse.medium.com/performance-best-practices-in-c-b85a47bdd93a
