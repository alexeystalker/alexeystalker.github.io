---
share: true
tags:
 - NET/SignalR
---
# Применяем расширенные настройки транспорта
Теперь заменим строку с `app.MapHub` на следующее.
```csharp
app.MapHub<LearningHub>("/learningHub", options =>
{
	options.Transports = HttpTransportType.WebSockets | HttpTransportType.LongPolling;
	options.CloseOnAuthenticationExpiration = true;
	options.ApplicationMaxBufferSize = 65_536;
	options.TransportMaxBufferSize = 65_536;
	options.MinimumProtocolVersion = 0;
	options.TransportSendTimeout = TimeSpan.FromSeconds(10);
	options.WebSockets.CloseTimeout = TimeSpan.FromSeconds(3);
	options.LongPolling,PollTimeout = TimeSpan.FromSeconds(10);

	Console.WriteLine($"Authorization data items: {options.AuthorizationData.Count}");
});
```
Это настройки транспорта, которые можно изменить. Разберем подробнее[^1]:
- `Transports` - указываем, какие транспортные механизмы доступны. По умолчанию доступны вебсокеты, SSE (server-sent events) и long-polling запросы. В примере мы указали, что можно использовать только вебсокеты и long-polling. Объединяются значения через побитовое ИЛИ, по принципу флагов;
- `CloseOnAuthenticationExpiration` - нужно ли закрывать соединение после окончания действия аутентификации. Если `true` - соединение закроется, если `false` - соединение будет оставаться открытым;
- `ApplicationMaxBufferSize` - ~~размер буфера *уровня приложения*, по умолчанию 65Кб~~ размер буфера данных, отправляемых приложением; после заполнения буфера активизируется логика замедления (backpressure);
- `TransportMaxBufferSize` - ~~размер буфера *уровня транспорта*, по умолчанию 65Кб~~ размер буфера данных, считываемых приложением; после заполнения буфера активизируется замедление (backpressure);
- `MinimumProtocolVersion` - минимальная версия протокола, поддерживаемая сервером. Если задано значение 0, принимается любая версия протокола;
- `TransportSendTimeout` - таймаут выполнения отправки. После превышения этого периода соединение закрывается.

Специфические настройки вебсокетов и лонгполлинга можно найти в документации.


[^1]:[Документация](https://learn.microsoft.com/ru-ru/dotnet/api/microsoft.aspnetcore.http.connections.httpconnectiondispatcheroptions?view=aspnetcore-6.0)