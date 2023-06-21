---
share: true
tags:
 - NET/SignalR
---
# Настраиваем хаб SignalR
## Добавляем хаб SignalR в проект
Следуя соглашениям MVC, добавим в наш проект папку **Hubs**. Затем добавим в эту папку новый файл **LearningHub.sc** со следующим содержимым:
```csharp
using Microsoft.AspNetCore.SignalR;
namespace SignalRServer.Hubs;

public class LearningHub : Hub
{
	public async Task BroadcastMessage(string message)
	{
		await Clients.All.SendAsync("RecieveMessage", message);
	}
	public override async Task OnConnectedAsync()
	{
		await base.OnConnectedAsync();
	}
	public override async Task OnDisconnectedAsync(Exception? exception)
	{
		await base.OnDisconnectedAsync(exception);
	}
}
```
## Обзор хаба SignalR
Прежде всего отметим,  что класс хаба должен быть унаследован от класса `Hub`. Далее, в хабе мы добавили метод `BroadcastMessage` с параметром типа `string`. Это пример метода, который смогут вызывать клиенты, подключившиеся к хабу. Он должен возвращать тип `Task`, и может иметь некоторое количество методов любых сериализуемых типов.
Итак, метод принимает строку, затем пересылает её всем клиентам, включая того, который вызвал метод. Для этого используется свойство `Clients`[^2]. Для выполнения отправки используется метод `SendAsync`.
Далее идут перегрузки методов `OnConnected` и `OnDisconnected`, при этом `OnDisconnected` имеет nullable параметр `Exception` на случай, если разрыв соединения произошёл из-за ошибки.
## Добавляем хаб как конечную точку
Чтобы добавить наш хаб как конечную точку (endpoint) в наше приложение, нужно:
- добавить сервисы SignalR
	```csharp
	builder.Services.AddSignalR();
	```
- добавить хаб как конечную точку по пути `/learningHub`[^1]
	```csharp
	app.MapHub<LearningHub>("/learningHub");
	```

	[^1]:Здесь показан простой маршрут, однако можно использовать и [[route-template|шаблоны маршрута]]. [[access-routing-values-from-httpcontext|Вот как извлекать из них значения]].
	[^2]: Подробнее [[ch-5-broadcasting-messages-to-all-clients|тут]].