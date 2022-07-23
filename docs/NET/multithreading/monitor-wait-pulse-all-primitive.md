---
tags: [NET/multithreading]
share: true
---
# Monitor.Wait / Monitor.Pulse / Monitor.PulseAll
Пара методов (Pulse / PulseAll считаем за один метод), предназначенных для временного отпускания блокировки с последующим взятием обратно. Все нижеперечисленные методы должны вызываться из [[lock-primitive|синхронизированного кода]].
Заметим, что ссылки на текущий блокирующий поток, *очередь готовности* и *очередь ожидания* хранятся в объекте блокировки.
## Monitor.Wait
```csharp
[System.Runtime.Versioning.UnsupportedOSPlatform("browser")]
public static bool Wait (object obj, int millisecondsTimeout, bool exitContext);
```
Перегрузки: `Wait(object)`, `Wait(object, int)`, `Wait(object, TimeSpan)`, `Wait(object, TimeSpan, bool)`.
Параметры: объект блокировки, таймаут в миллисекундах или таймспан, флаг выхода из контекста синхронизации.
Возвращаемое значение: `true`, если блокировка взята до таймаута, `false`, если после. Метод не завершается до момента повторного взятия блокировки.
## Monitor.Pulse
```csharp
public static void Pulse (object obj);
```
Уведомляет следующий метод, ожидающий блокировку. Уведомленный метод перемещается в *очередь готовности*. После отпускания методом блокировки блокировку получает первый метод из очереди готовности (возможно, не метод, получивший `Pulse` сигнал!).
> [!Important] Важно!
> `Monitor` не хранит информацию о том, был ли взведен `Pulse`, поэтому, если `Pulse` был взведен до вызова `Wait`, можно получить deadlock.
## Monitor.PulseAll
```csharp
public static void PulseAll (object obj);
```
Уведомляет ВСЕ методы, ждущие блокировку объекта. После уведомления все методы перемещаются в *очередь готовности*, и первый из них получает блокировку после отпускания ее текущим методом.


## Ссылки
https://docs.microsoft.com/en-us/dotnet/api/system.threading.monitor.wait?view=net-6.0
https://docs.microsoft.com/en-us/dotnet/api/system.threading.monitor.pulse?view=net-6.0