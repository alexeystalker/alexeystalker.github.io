---
share: true
tags:
 - NET/SignalR
---
# Настраиваем .NET клиент
## Делаем клиент к SignalR в консольном приложении
Создадим консольное приложение:
```bash
dotnet new console -o DotnetClient
```
и добавим к нему NuGet-пакет с клиентом SignalR:
```bash
dotnet add package Microsoft.AspNetCore.SignalR.Client
```
Затем, нам нужно реализовать логику использования клиента.
Сперва удалим всё содержимое файла **Program.cs** и заменим его сперва следующим:
```csharp
using Microsoft.AspNetCore.SignalR.Client;
 
Console.WriteLine("Please specify the URL of SignalR Hub");
var url = Console.ReadLine();
//Создаем объект HubConnection
var hubConnection = new HubConnectionBuilder().WithUrl(url).Build();
```
Затем настроим обработку события `ReceiveMessage` - каждый раз, как сервер будет вызывать этот метод, будем выводить сообщение в консоль:
```csharp
hubConnection.On<string>("ReceiveMessage",
						message => Console.WriteLine($"SignalR Hub Message: {message}"));
```
Теперь сделаем так, что каждое введённое нами в консоли сообщение отправлялось на сервер в качестве параметра метода `BroadcastMessage` до тех пор, пока мы не введём exit:
```csharp
try
{
	await hubConnection.StartAsync();
	
	while (true)
	{
		var message = string.Empty;
		
		Console.WriteLine("Please specify the action:");
		Console.WriteLine("0 - broadcast to all");
		Console.WriteLine("exit - Exit the program");
		
		var action = Console.ReadLine();
		if(action = "exit")
			break;
		
		Console.WriteLine("Please specify the message");
		message = Console.ReadLine("");
		await hubConnection.SendAsync("BroadcastMessage", message);
	}
}
catch (Exception ex)
{
	Console.WriteLine(ex.Message);
	Console.WriteLine("Press any key to exit...");
	Console.ReadKey();
	return;
}
```
Также здесь приведена базовая логика обработки исключений.
## Запускаем клиент
Запускаем наш клиент при помощи команды `dotnet run`. Далее нам надо указать URL нашего хаба. В файле **launchSettings.json** нашего серверного приложения, в разделе *applicationUrl* указан URL нашего приложения, в частности порт. Например, пусть это будет `https://localhost:7128`. Тогда URL нашего хаба будет `https://localhost:7128/learningHub`. Его и надо ввести, чтобы подключиться к хабу.