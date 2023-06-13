---
share: true
tags:
 - NET/SignalR
---
# Работа с группами клиентов
Класс `Hub` содержит свойство `Groups`, с помощью которого можно добавлять клиентов в группу или удалять из неё. Под "группой" понимается отношение "один ко многим" между произвольной строкой ("именем") и коллекцией идентификаторов подключения. У свойства хаба `Clients` есть метод `Group()`, который позволяет отправлять сообщения всем клиентам в группе с указанным именем.
Давайте посмотрим, как можно использовать группы.
## Использование групп внутри хаба
Добавим к нашему хабу следующие методы:
```csharp
public async Task SendToGroup(string groupName, string message)
{
	await Clients.Group(groupName).ReceiveMessage(GetMessageToSend(message));
}

public async Task AddUserToGroup(string groupName)
{
	await Groups.AddToGroupAsync(Context.ConnectionId, groupName);
	await Clients.Caller.ReceiveMessage($"Current user added to {groupName} group");
	await Clients.Others.ReceiveMessage($"User {Context.ConnectionId} added to {groupName} group");
}

public async Task RemoveUserFromGroup(string groupName)
{
	await Groups.RemoveFromGroupAsync(Context.ConnectionId, groupName);
	await Clients.Caller.ReceiveMessage($"Current user removed from {groupName} group");
	await Clients.Others.ReceiveMessage($"User {Context.ConnectionId} removed from {groupName} group");	
}
```
Здесь мы, кроме добавления в группу или удаления из нее отправляем сообщения о том, что пользователь добавлен в группу или покинул её, причём сообщения для клиента, вызвавшего метод и других клиентов будут различаться.
Также мы дополним методы `OnConnectedAsync` и `OnDisconnectedAsync` — будем добавлять подключившегося клиента в группу `HubUsers` и удалять из неё отключившегося — просто для демонстрации возможности автоматического добавления в группы.
## Используем группы на клиентах
Как обычно, сперва добавим разметку для JavaScript клиента.
```html
<div class="control-group">
	<div>
		<label for="group-message">Message</label>
		<input type="text" id="group-message" name="group-message" />
	</div>
	<div>
		<label for="group-for-message">Group Name</label>
		<input type="text" id="group-for-message" name="group-for-message" />
	</div>
	<button id="btn-group-message">Send to Group</button>
</div>
<div class="control-group">
	<div>
		<label for="group-to-add">Group Name</label>
		<input type="text" id="group-to-add" name="group-to-add" />
	</div>
	<button id="btn-group-add">Add User to Group</button>
</div>
<div class="control-group">
	<div>
		<label for="group-to-remove">Group Name</label>
		<input type="text" id="group-to-remove" name="group-to-remove" />
	</div>
	<button id="btn-group-remove">Remove User from Group</button>
</div>
```
Теперь — обработчики кликов
```js
$('#btn-group-message').click(function () {
	var message = $('#group-message').val();
	var group = $('#group-for-message').val();
	connection.invoke("SendToGroup", group, message).catch(err => console.error(err.toString()));
});
$('#btn-group-add').click(function () {
	var group = $('#group-to-add').val();
	connection.invoke("AddUserToGroup", group).catch(err => console.error(err.toString()));
});
$('#btn-group-remove').click(function () {
	var group = $('#group-to-remove').val();
	connection.invoke("RemoveUserFromGroup", group).catch(err => console.error(err.toString()));
});
```
Теперь дополним код нашего .NET клиента:
```csharp
//...
var groupName = string.Empty;
//...
Console.WriteLine("4 - send to a group");
Console.WriteLine("5 - add user to a group");
Console.WriteLine("6 - remove user from a group");
//...
if(action != "5" && action != "6")
{
	Console.WriteLine("Please specify the message:");
	message = Console.ReadLine();
}
if(action == "4" || action == "5" || action == "6")
{
	Console.WriteLine("Please specify the group name");
	groupName = Console.readLine();
}

switch (action)
{
//...
	case "4":
		hubConnection.SendAsync("SendToGroup", groupName, message).Wait();
		break;
	case "5":
		hubConnection.SendAsync("AddUserToGroup", groupName).Wait();
		break;
	case "6":
		hubConnection.SendAsync("RemoveUserFromGroup", groupName).Wait();
		break;
//...
}
//...
```