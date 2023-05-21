---
share: true
tags: [NET/async,NET/CancellationToken,NET/CancellationTokenSource]
---
# Подписки (subscriptions) в CancellationTokenSource
Когда мы отменяем таск через CancellationTokenSource, все подписки выполняются в текущем потоке. Это может приводить к дедлокам или неожиданным задержкам.
```csharp
var cts = new CancellationTokenSource();
cts.Token.Register(() => Thread.Sleep(5000));
cts.Cancel(); // This call will block during 5 seconds
```
Поэтому, если мы не можем ожидать выполнения ВСЕХ подписок в текущем потоке, нужно обернуть вызов `cts.Cancel()` в `Task.Run`
## Ссылки
https://kevingosse.medium.com/performance-best-practices-in-c-b85a47bdd93a
