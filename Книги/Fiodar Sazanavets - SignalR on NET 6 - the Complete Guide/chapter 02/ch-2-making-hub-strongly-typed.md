---
share: true
tags:
 - NET/SignalR
---
# Делаем хаб типизированным
В [[ch-2-setting-up-signalr-hub|прошлом разделе]] мы использовали обычную строку для указания на клиентское событие, которое нам нужно выполнить. Теперь, для того чтобы исключить появления разного рода трудноуловимых ошибок, связанных с опечатками и устареванием строковых констант, добавим формальное описание контракта клиента, другими словами - интерфейс.
```csharp
public interface ILearningHubClient
{
	Task ReceiveMessage(string message);
}
```
Мы указали имя клиентского события в качестве имени метода. Теперь модифицируем сам хаб:
```csharp
public class LearningHub : Hub<ILearningHubClient>
{
	public async Task BroadcastMessage(string message)
	{
		await Clients.All.ReceiveMessage(message);
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
Здесь мы унаследовали наш хаб от дженерика `Hub<>`, и это позволило нам вызвать метод у клиента напрямую, без необходимости использовать `SendAsync()` и строки.