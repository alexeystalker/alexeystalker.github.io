---
share: true
tags:
 - NET/SignalR
---
# Отправка сообщений конкретным клиентам
## Отправка сообщений самому себе
В [[ch-5-broadcasting-messages-to-all-clients|предыдущем разделе]] мы увидели, как отправить сообщение всем клиентам, кроме клиента, вызвавшего метод. Теперь поступим обратным образом — отправим сообщение только клиенту, вызвавшему метод. Для этого воспользуемся специальным свойством — `Clients.Caller`.
Добавим метод в наш хаб `LearningHub`:
```csharp
public async Task SendToCaller(string message)
{
	await Clients.Caller.ReceiveMessage(message);
}
```
Теперь добавим разметку к нашему JavaScript клиенту:
```html
<div class="control-group">
	<div>
		<label for="self-message">Message</label>
		<input type="text" id="self-message" name="self-message" />
	</div>
	<button id="btn-self-message">Send to Self</button>
</div>
```
И теперь добавим обработчик клика кнопки:
```js
$('#btn-self-message').click(function () {
	var message = $('#self-message').val();
	connection.invoke("SendToCaller", message).catch(err => console.error(err.ToString()));
});
```
Также дополним код нашего .NET клиента вариантом действия `2` (полный код см. в предыдущем разделе):
```csharp
//...
Console.WriteLine("2 - send to self");
//...
switch (action)
{
//...
	case "2":
		await hubConnection.SendAsync("SendToCaller", message);
//...
}
//...
```
Теперь, если мы отправим сообщение с какого-нибудь из клиентов, только он его и получит в ответ.
## Отправка сообщений другому клиенту
У хаба SignalR есть свойство под названием `Context`. Это свойство содержит метаданные текущего подключения. В частности — идентификатор подключения `ConnectionId`. Если вы знаете идентификатор подключения конкретного клиента, вы можете отправлять сообщения этому клиенту.
### Изменяем код хаба
Сперва добавим метод, подготавливающий сообщение к отправке клиенту:
```csharp
private string GetMessageToSend(string originalMessage)
{
	return $"User connection id: {Context.ConnectionId}. Message: {originalMessage}";
}
```
Далее заменим все вызовы клиентского метода `ReceiveMessage(message)` на `ReceiveMessage(GetMessageToSend(message))`, таким образом мы сможем передать наш идентификатор подключения другим клиентам. Теперь добавим метод хаба:
```csharp
public async Task SendToIndividual(string connectionId, string message)
{
	await Clients.Client(connectionId).ReceiveMessage(GetMessageToSend(message));
}
```
> [!Note] Примечание
> Помимо метода `Client()` у свойства `Clients` есть метод `Clients(IReadOnlyList<string> connectionIds)`, который позволяет отправить сообщения клиентам с идентификаторами подключений, перечисленными в параметре

### Изменяем код клиентов
Сперва дополним разметку JavaScript клиента:
```html
<div class="control-group">
	<div>
		<label for="individual-message">Message</label>
		<input type="text" id="individual-message" name="individual-message" />
	</div>
	<div>
		<label for="connection-for-message">User connection id:</label>
		<input type="text" id="connection-for-message" name="connection-for-message" />
	</div>
	<button id="btn-individual-message">Send to Specific User</button>
</div>
```
Затем — обработчик клика:
```js
$('#btn-individual-message').click(function () {
	var message = $('#individual-message').val();
	var connectionId = $('#connection-for-message').val();
	connection.invoke("SendToIndividual", connectionId, message).catch(err => console.error(err.toString()));
});
```
И, наконец, дополним код .NET клиента:
```csharp
//...
Console.WriteLine("3 - send to individual");
//...
switch (action)
{
//...
	case "3":
		Console.WriteLine("Please specify the connection id:");
		var connectionId = Console.ReadLine();
		await hubConnection.SendAsync("SendToIndividual", connectionId, message);
//...
}
//...
```
### Смотрим на индивидуальные сообщения в действии
Сперва отправим широковещательные сообщения со всех клиентов — так мы получим их идентификаторы подключений. После чего можно отправить сообщение любому клиенту на выбор.

---
Как видно, способ с отправкой индивидуального сообщения несколько неудобен тем, что нужно обязательно узнать идентификатор, прежде чем отправлять сообщения. К тому же, если произойдёт разрыв связи, и клиент переподключится — его идентификатор подключения изменится, и наше сообщение с прежним идентификатором просто не дойдёт.
Чтобы решить эту проблему, воспользуемся встроенным [[ch-5-working-with-client-groups|механизмом групп]].