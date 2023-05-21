---
share: true
tags: [NET/performance,NET/antipatterns]
---
# Шаблоны кода на CSharp, которых следует избегать
1. [[wait-sync|Синхронно ждать асинхронный код]];
2. [[async-void|async void]];
3. [[return-await|Ненужный return await]];
4. Использование `ConcurrentBag<T>` - тормозит; стараемся использовать `ConcurrentQueue<T>`;
5. Использование `ReaderWriterLock<T>` или `ReaderWriterLockSlim<T>` - тормозит. Лучше использовать `Monitor` под lock-ом;
6. [[lambda-vs-method-group|Избегать MethodGroup для статических методов]];
7. [[enumequalsboxing|Делать Enum.Equals]];
8. [[equality-structs|Использовать дефолтные]] методы сравнения в структурах;
9. [[structs-in-interfaces|Использовать структуры в интерфейсах]];
10. [[cts-register-cancel|Регистрировать долгие]] таски в CancelTokenSource;
11. [[tcs-runconts|Не использовать]] параметр `TaskCreationsOptions.RunContinuationsAsynchronously` при использовании `TaskCompletionSource`;
12. [[task-factory-start-new|Использовать]] `Task.Factory.StartNew` без причины;
## Ссылки
https://kevingosse.medium.com/performance-best-practices-in-c-b85a47bdd93a
