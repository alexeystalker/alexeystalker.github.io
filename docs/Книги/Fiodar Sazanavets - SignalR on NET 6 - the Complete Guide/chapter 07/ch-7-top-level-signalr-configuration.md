---
share: true
tags:
 - NET/SignalR
---
# Настройка SignalR верхнего уровня
В файле **Program.cs** изменим команду добавления SignalR на следующее:
```csharp
builder.Services.AddSignalR(hubOptions => 
{
	hubOptions.KeepAliveInterval = TimeSpan.FromSeconds(10);
	hubOptions.MaximumReceiveMessageSize = 65_536;
	hubOptions.HandshakeTimeout = TimeSpan.FromSeconds(15);
	hubOptions.MaximumParallelInvocationsPerClient = 2;
	hubOptions.EnableDetailedErrors = true;
	hubOptions.StreamBufferCapacity = 15;

	if (hubOptions.SupportedProtocols is not null)
	{
		foreach (var protocol in hubOptions.SupportedProtocols)
			Console.WriteLine($"SignalR supports {protocol} protocol");
	}
});
```
В этом коде показаны все возможные верхнеуровневые настройки:
- `KeepAliveInterval` — интервал между ping-запросами к подключённым клиентам. По умолчанию 15 секунд;
- `MaximumReceiveMessageSize` — максимальный размер сообщения, который клиент может отправить хабу. По умолчанию 32 кБ. Есть смысл увеличивать размер, если предполагается отправлять сообщения большого объема, и есть смысл уменьшить размер для повышения производительности;
- `HandshakeTimeout` — когда клиент устанавливает соединение с хабом SignalR он инициирует процедуру рукопожатия (handshake). Эта настройка определяет, как долго сервер должен ждать ответа от клиента в рамках этой процедуры, прежде чем соединение будет признано разорваным. По умолчанию 15 секунд;
- `MaximumParallelInvocationsPerClient` — настройка определяет, сколько методов сервера могут быть вызваны каждым клиентом одновременно; остальные вызовы будут поставлены в очередь. По умолчанию 1;
- `EnableDetailedErrors` — определяет, будут ли клиенты получать детализированные сообщения об ошибках при выполнении серверных методов, в частности, сообщения внутренних исключений. По умолчанию false;
- `StreamBufferCapacity` — настройка отвечает за количество элементов, которые могут быть загружены в поток [[ch-6-client-streaming-in-signalr|"Клиент к серверу"]]. После достижения порога вызов блокируется до обработки существующих элементов. По умолчанию 10;
- `SupportedProtocols` — коллекция имён поддерживаемых протоколов, например `json` или `messagepack`. По умолчанию активирован только JSON. [[ch-7-pros-and-cons-of-messagepack-protocol|MessagePack]] или, например, [[signalr-newtonsoft-json|NewtonsoftJson]] должны быть явно активированы.