---
aliases:
 - lock
tags: [NET/multithreading]
share: true
---
# lock (Monitor.Enter / Monitor.Exit)
Ключевое слово `lock` используется для реализации идеи критической секции.
Использует ссылочный объект в качестве объекта синхронизации.
Критическая секция оформляется следующим образом:
```csharp
lock(syncObj)
{
	/*код критической секции*/
}
```
Является синтаксическим сахаром над следующей конструкцией:
```csharp
Monitor.Enter(syncObj);
try
{
	/*код критической секции*/
}
finally
{
	Monitor.Exit(syncObj);
}
```
Желательно не использовать внутри такой критической секции асинхронные методы (с `await`), так как `Monitor.Exit()` должен выполняться на том же потоке, что и `Monitor.Enter()`, иначе будет выброшено исключение `SynchronizationLockException`. Исключение из этого правила составляет случай, когда существует контекст синхронизации, и управление вернется в тот же поток (например, UI-поток WinForms или WPF).
## Monitor.Enter(Object, Boolean)
`Monitor.Enter(Object, Boolean)` — перегрузка метода Enter, помимо взятия блокировки атомарно записывающая во второй параметр состояние блокировки. 
## Monitor.Enter / Monitor.Exit и ValueType
Если передавать в `Monitor.Enter()`переменную `ValueType` типа, она подвергнется боксингу. При этом каждый раз создается новый ссылочный объект, и, соответственно, блокировка не выполнится. То же в случае с `Monitor.Exit()` — так как в метод передалась ссылка на новую переменную, `Monitor.Exit()` выбросит исключение `SynchronizationLockException`.

## Ссылки
https://docs.microsoft.com/en-us/dotnet/api/system.threading.monitor.enter?view=net-6.0
https://docs.microsoft.com/en-us/dotnet/api/system.threading.monitor.exit?view=net-6.0
[Monitor class overview](https://docs.microsoft.com/en-us/dotnet/api/system.threading.monitor?view=net-6.0#the-monitor-class-an-overview)