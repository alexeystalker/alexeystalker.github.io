---
share: true
tags: 
  - NET/memory-leak
  - NET/CancellationTokenSource
  - NET/CancellationToken
---
# CancellationTokenSource и утечка памяти
В `CancellationTokenSource` запоминаются (до вызова `Cancel()`) все объекты, которые регистрируют собственный метод отмены. Вывод - если нужно регистрировать свой метод для каждого объекта в цикле, нужно создавать свой `CancellationTokenSource`  через `CreateLinkedTokenSource()`, то есть
```csharp
var linkedCts = CancellationTokenSource.CreateLinkedTokenSource(parentCts.Token);
```
## Ссылки
[Статья на Хабре](https://habr.com/ru/company/tinkoff/blog/546604/#habracut)
https://docs.microsoft.com/en-us/dotnet/api/system.threading.cancellationtokensource.createlinkedtokensource?view=net-6.0