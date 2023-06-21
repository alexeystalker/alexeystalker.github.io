---
share: true
tags:
 - NET/SignalR
 - NET/websocket
---
# Настраиваем чистый Websocket клиент
Предположим, что нам нужно подключиться к хабу SignalR из приложения, написанного на языке, для которого нет официальной библиотеки клиента SignalR. В этом случае можно использовать чистый Websocket клиент, реализация которого с большой вероятностью уже есть для вашего языка.
Для простоты используем клиента Websocket для .NET.
## Настраиваем клиент WebSocket
Создадим новый проект консольного приложения. Так как в нем не будет никаких прямых зависимостей от наших существующих проектов, его не обязательно включать в общее с ними решение.
```bash
dotnet new console -o WebSocketClient
```
Никаких дополнительных пакетов добавлять не нужно, клиент вебсокетов включён в стандартную библиотеку.
Теперь удалим содержимое файла **Program.cs** и добавим нужные юзинги и код для получения адреса нашего хаба.
```csharp
using System.Net.Websockets;
using System.Text;

Console.WriteLine("Please specify the URL of SignalRHub with WS/WSS protocol");
var url = Console.Readline();
```
> [!important] Важно!
> Вебсокеты используют схему WS/WSS в URL вместо HTTP/HTTPS, однако тот же нижележащий протокол TCP/IP. Поэтому адрес и порт вашего приложения останутся теми же самыми.  Таким образом, необходимо указать адрес вида `wss://{base URL}/learningHub` вместо `https://{base URL}/learningHub`.

Теперь нам нужно подключиться к Websocket и отправить начальный запрос нашему хабу. Это будет рукопожатие (handshake), ожидаемое хабом, с определением протокола и его версии.
```csharp
try
{
	var ws = new ClientWebSocket();
	await ws.ConnectAsync(new Uri(url), CancellationToken.None);
	var handshake = new List<byte>(
		Encoding.UTF8.GetBytes(@"{""protocol"":""json"", ""version"":1}"))
	{
		0x1e
	};
	await ws.SendAsync(
		new ArraySegment<byte>(handshake.ToArray()),
		WebSocketMessage.Type.Text,
		true,
		CancellationToken.None);
	
	Console.WriteLine("WebSockets connection established");
	await ReceiveAsync(ws);
}
catch (Exception ex)
{
	Console.WriteLine(ex.Message);
	Console.WriteLine("Press any key to exit...");
	Console.ReadKey();
	return;
}
```
Теперь нам нужно реализовать метод `ReceiveAsync`. Вот он:
```csharp
static async Task ReceiveAsync(ClientWebSocket ws)
{
	var buffer = new byte[4096];

	try
	{
		while (true)
		{
			var result = await ws.ReceiveAsync(
				new ArraySegment<byte>(buffer),
				CancellationToken.None);
			if (result.MessageType == WebSocketMessageType.Close)
			{
				await ws.CloseOutputAsync(
					WebSocketCloseStatus.NormalClosure,
					string.Empty,
					CancellationToken.None);
				break;
			}
			else
			{
				Console.WriteLine(Encoding.Default.GetString(Decode(buffer)));
				buffer = new byte[4096];
			}
		}
	}
	catch (Exception ex)
	{
		Console.WriteLine(ex.Message);
		Console.WriteLine("Press any key to exit...");
		Console.ReadKey();
		return;
	}
}
```
Тут мы печатаем все получаемые сообщения до тех пор, пока не получим сообщение о закрытии соединения. Однако, как мы видим, для печати мы обрабатываем буфер в методе `Decode`. Этот метод обрезает байты, оставшиеся незаполненными после получения сообщения.
```csharp
static byte[] Decode(byte[] packet)
{
	var i = packet.Length - 1;
	while (i >= 0 && packet[i] == 0)
	{
		--i;
	}

	var temp = new byte[i + 1];
	Array.Copy(packet, temp, i + 1);
	return temp;
}
```
## Запускаем клиент WebSocket
Теперь запустим наш клиент, указав адрес хаба в виде  `wss://{base URL}/learningHub`.  Первым делом мы получим от сервера пустое сообщение — так и должно быть, это часть процедуры рукопожатия (handshake). 
Далее, когда мы начнём получать наши широковещательные (broadcast) сообщения, мы увидим, что приходят они в виде JSON, с другими полями, что-то вроде такого:
```json
{"type":1,"target":"ReceiveMessage","arguments":["message from the browser"]}
```
Здесь мы видим, что есть поля `type`, `target` и `arguments` — так SignalR понимает, какой метод мы вызываем и какие значения параметров передаём.
Затем, когда приложение проработает какое-то время, нам придёт сообщение такого вида:
```json
{"type":6}
```
Здесь мы видим тип сообщения 6 — это сообщение типа heartbeat, с его помощью проверяется, что соединение не разорвано. 1 — значение поля `type` для обычных сообщений[^1].

[^1]: Другие значения поля `type` можно найти, посмотрев [код библиотеки](https://github.com/dotnet/aspnetcore/tree/main/src/SignalR)