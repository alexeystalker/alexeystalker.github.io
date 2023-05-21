---
share: true
tags:
 - NET/logging
---
# Использование областей журналирования для добавления дополнительных свойств в сообщения журнала
*Области журналирования (scopes)* позволяют связывать одни и те же свойства с каждым сообщением, записанным внутри области.

Область журналирования можно создать путём вызова `ILogger.BeginScope<T>(T state)` внутри блока `using`:
```csharp
_logger.LogInformation("No, I don't have scope");

using(_logger.BeginScope("Scope value"))
using(_logger.BeginScope(new Dictionary<string, object> { ["CustomValue1"] = 12345 }))
{
	_logger.LogInformation("Yes, I have the scope!");
}
_logger.LogInformation("No, I lost it again");
```
Состояние области может быть любым объектом — целым числом, строкой или словарём. Реализация поставщика должна решать, как обрабатывать состояние, предоставленное в `BeginScope()`.
> [!tip] Совет
> Наиболее часто области используются для добавления дополнительных пар “ключ-значение”. При использовании [[datalust-seq|Seq]] и [[library-serilog|Serilog]] нужно передать `Dictionary<string, object>` в качестве объекта состояния[^1].

[^1]: Создатель Seq и Serilog Николас Блюмхардт приводит примеры и аргументы в пользу такого подхода в статье [Семантика ILogger.BeginScope()](https://nblumhardt.com/2016/11/ilogger-beginscope/)