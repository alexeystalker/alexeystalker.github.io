---
share: true
tags:
 - NET/SignalR
---
# Рассылка сообщений всем клиентам
В предыдущих главах мы уже рассылали сообщения всем подключённым клиентам при помощи свойства хаба `Clients.All`. Но что, если мы захотим исключить отправителя из списка получателей?
В SignalR это можно сделать, используя свойство `Clients.Other` вместо `Clients.All`.
В файле **LearningHub.cs** добавим к нашему хабу следующий метод:
```csharp
public async Task SendToOthers(string message)
{
	await Clients.Others.ReceiveMessage(message);
}
```
Теперь добавим возможность вызывать этот метод к нашим клиентам.
## Вносим изменения в клиент JavaScript
сперва изменим наш клиент JavaScript. Прежде всего добавим новые элементы управления в разметку страницы. Откроем файл **Views/Home/Index.cshtml**, найдём элемент `div` со значением атрибута `class="col-md-4"` и добавим в него следующее:
```html
<div class="control-group">
	<div>
		<label for="others-message">Message</label>
		<input type="text" id="others-message" name="others-message" />
	</div>
	<button id="btn-others-message">Send to Others</button>
</div>
```
Теперь нам надо добавить соответствующий код JavaScript. Откроем файл **wwwroot/js/site.js** и добавим обработчик нажатия:
```js
$('#btn-others-message').click(function () {
	var message = $('#others-message').val();
	connection.invoke("SendToOthers", message).catch(err => console.error(err.toString()));
});
```
## Вносим изменения в клиент .NET
В проекте DotnetClient откроем файл **Program.cs** и заменим содержимое блока `try` следующим:
```csharp
await hubConnection.StartAsync();

var running = true;

while (running)
{
	var message = string.Empty;
	
	Console.WriteLine("Please specify the action:");
	Console.WriteLine("0 - broadcast to all");
	Console.WriteLine("1 - send to others");
	Console.WriteLine("exit - exit the program");

	var action = Console.ReadLine();

	Console.WriteLine("Please specify the message");
	message = Console.ReadLine();

	switch (action)
	{
		case "0":
			await hubConnection.SendAsync("BroadcastMessage", message);
			break;
		case "1":
			await hubConnection.SendAsync("SendToOthers", message);
			break;
		case "exit":
			running = false;
			break;
		default:
			Console.WriteLine("Invalid action specified");
			break;
	}
}
```
Теперь у нас есть два возможных действия - отправить сообщение всем (и себе), или другим подключённым клиентам.
> [!tip]- Примечание от меня
> Как некоторые обратили внимание, код клиента написан несколько странно - даже если мы ввели команду `exit`, нас всё равно заставляют ввести текст сообщения. Кажется, рациональнее было бы при получении команды `exit` сразу же закончить цикл. Однако, оставим это на совести автора.