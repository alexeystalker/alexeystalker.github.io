---
share: true
tags:
 - NET/SignalR
---
# Стриминг с клиента в SignalR
В данном случае мы открываем поток на стороне клиента, отправляем сообщения до тех пор, пока они не закончатся или поток не будет закрыт. Сервер получает сообщения в том же порядке, в каком они были отправлены и в рамках одного контекста вызова.
При нормальных обстоятельствах, (иными словами, если не произошло ошибок) поток контролируется клиентом. Сервер может принудительно закрыть поток, если что-то пошло не так, но нормальное закрытие (normal closure) инициируется клиентом.
Сперва модифицируем код клиентов.
## Добавляем стриминг к клиенту JavaScript
В файле **wwwroot/js/site.js** заменим обработчик клика для селектора `#btn-broadcast` следующим кодом:
```js
$('#btn-broadcast').click(function () {
	var message = $('#broadcast').val();

	if (message.includes(';')) {
		var messages = message.split(';');
		var subject = new SignalR.Subject();
		connection.send("BroadcastStream", subject),catch(err => console.error(err.toString()));
		for (var i = 0; i < messages.length; i++) {
			subject.next(messages[i]);
		}
		subject.complete();
	} else {
		connection.invoke("BroadcastMessage", message).catch(err => console.error(err.toString()));
	}
});
```
Если в строке сообщения не встречается `;`, то мы отправляем сообщение, как раньше. В противном случае мы разбиваем строку в массив по этому символу, вызываем метод `BroadcastStream`, открывая для этого метода поток. Затем отправляем в этом потоке наш массив поэлементно. Затем закрываем поток.
Клиентский поток здесь управляется при помощи типа `Subject`. Объект этого типа предоставляет абстракцию открытого потока. Создав объект мы открываем поток, затем передаём его методу, далее отправляем элементы массива через `next()`, и в конце закрываем поток при помощи `complete()`.
## Добавляем стриминг к клиенту .NET
В файл **Program.cs** нашего клиента добавляем юзинг:
```csharp
using System.Threading.Channels
```
Затем поменяем `case 0` в свиче на следующее:
```csharp
case 0:
	if (message?.Contains(';') ?? false)
	{
		var channel = Channel.CreateBounded<string>(10);
		await hubConnection.SendAsync("BroadcastStream", channel.Reader);
		
		foreach (var item in message.Split(';'))
		{
			await channel.Writer.WriteAsync(item);
		}
		channel.Writer.Complete();
	}
	else
	{
		hubConnection.SendAsync("BroadcastMessage", message).Wait();
	}
	break;
```
Здесь происходит то же самое, что и в JavaScript клиенте. Объект класса `Channel` представляет собой абстракцию потока. `Reader` мы передаём хабу, а при помощи `Writer` отправляем сообщения. Затем закрываем поток.
## Добавляем слушателя клиентских потоков к хабу SignalR
У метода хаба `BroadcastStream` такой код:
```csharp
public async Task BroadcastStream(IAsyncEnumerable<string> stream)
{
	await foreach(var item in stream)
	{
		await Clients.Caller.ReceiveMessage($"Server received {item}");
	}
}
```
За представление потока здесь отвечает асинхронный поток `IAsyncEnumerable`. На каждый переданный элемент мы отправляем вызвавшему метод клиенту подтверждение.