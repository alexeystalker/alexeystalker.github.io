---
share: true
tags: [NET/async]
---
# async void
Не должен использоваться где-либо, кроме (если очень нужно) обработчика события (event).
Основная причина - неловленное в этом методе исключение распространится на весь контекст синхронизации, что обычно приводит к падению всего приложения (если обернуть вызов метода в try/catch, исключение НЕ ПОЙМАЕТСЯ).
Если в методе нельзя возвратить Task (например, при реализации чужого интерфейса), необходимо перенести асинхронный код в другой метод и вызвать его оттуда, например
```csharp
interface IInterface
{
    void DoSomething();
}

class Implementation : IInterface
{
    public void DoSomething()
    {
        // переносим асинхронный код в другой метод
        _ = DoSomethingAsync();
    }

    private async Task DoSomethingAsync()
    {
        await Task.Delay(100);
    }
}
```

> [!tip]- Нюанс
> этот код не будет дожидаться выполнения `DoSomethingAsync()`
> Если нужно всё-таки дождаться выполнения, то тут всё-таки можно использовать `GetAwaiter().GetResult()`, но помним о [[wait-sync|последствиях]]. 

## Ссылки
https://docs.microsoft.com/ru-ru/archive/msdn-magazine/2013/march/async-await-best-practices-in-asynchronous-programming
