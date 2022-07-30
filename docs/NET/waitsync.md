---
share: true
tags: [NET/async,NET/threadpool]
---
# Ожидание асинхронного кода может привести к Threadpool starvation
Если злоупотреблять ожиданием асинхронного кода, можно выбрать весь тредпул и получить ситуацию голодания (Threadpool starvation). Это касается всех ожидателей, как-то `Task.Wait`, `Task.Result`, `Task.GetAwaiter().GetResult()`, `Task.WaitAny`, `Task.WaitAll` (последние не путать с `Task.WhenAny` и `Task.WhenAll`!!)
## Ссылки
https://medium.com/criteo-engineering/net-threadpool-starvation-and-how-queuing-makes-it-worse-512c8d570527

