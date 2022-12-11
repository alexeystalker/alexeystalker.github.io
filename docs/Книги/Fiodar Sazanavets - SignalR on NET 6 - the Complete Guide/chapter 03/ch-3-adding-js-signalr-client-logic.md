---
share: true
tags:
 - NET/SignalR
 - JavaScript
---
# Добавляем клиентскую логику
Наш клиент SignalR будет работать в браузере, поэтому нам нужно сделать страницу с контролами, которые позволят нам отправлять сообщения с клиента. Для этого возьмём файл **Views/Home/Index.cshtml** и заменим его содержимое следующим кодом
```razor
@{
	ViewData["Title"] = "Home Page";
}

<div class="row" style="padding-top: 50px;">
	<div class="col-md-4">
		<div class="control-group">
			<div>
				<label for="broadcast">Message</label>
				<input type="text" id="broadcast" name="broadcast" />
			</div>
			<button id="btn-broadcast">Broadcast</button>
		</div>
	<div>
	
	<div class="col-md-7">
		<p>SignalR Messages:</p>
		<pre id="signalr-message-panel"></pre>
	</div>
</div>
```
Также добавим немного стилей, чтобы этой страницей можно было пользоваться. Для этого откроем файл **wwwroot/css/site.css** и, не меняя существующего содержимого, добавим следующее
```css
.body-content {
	padding-left: 15px;
	padding-right: 15px;
}
.control-group {
	padding-top: 50px;
}
label {
	width: 100px;
}
#signalr-message-panel {
	height: calc(100vh - 200px);
}
```
Наконец, добавим немного кода Javascript, который будет выполняться по нажатию кнопок. Добавим в файл **wwwroot/js/site.js** следующий код
```js
const connection = new signalR.HubConnectionBuilder()
	.withUrl("/learningHub")
	.configureLogging(signalR.LogLevel.Information)
	.build();

connection.on("ReceiveMessage", (message) => {
	$('#signalr-message-panel').prepend($('<div />').text(message));
});

$('#btn-broadcast').click(function () {
	var message = $('#broadcast').val();
	connection.invoke("BroadcastMessage", message).catch(err => console.error(err.toString)));
});

async function start () {
	try {
		await connection.start();
		console.log('connected');
	} catch (err) {
		console.log(err);
		setTimeout(() => start(), 5000);
	}
};

connection.onclose(async () => {
	await start();
});

start();
```
Здесь мы сперва строим объект, отвечающий за подключение к серверу SignalR. Делаем это при помощи метода `HubConnectionBuilder` объекта `signalR`. Указываем URL при помощи метода `withUrl`. URL может быть как абсолютным, так и относительным, как в нашем случае. Далее указываем необходимый нам уровень логирования и строим объект (метод `build`).
После этого мы определяем обработчик события `ReceiveMessage`, определенного ранее в серверном коде.
Далее идёт обработчик нажатия на кнопку с `id="btn-broadcast"`, в котором вызываем серверный метод `BroadcastMessage` при помощи метода `connection.invoke`. Текст сообщения передаётся параметром, также добавили обработку ошибок.
Затем мы определяем функцию `start`, которая устанавливает соединение, делая повторные попытки, если что-то пошло не так, а также добавляем обработчик `onclose`, который переподключается, если соединение по каким-либо причинам закрылось.
И, наконец, вызываем `start` для начала работы.