---
share: true
tags:
 - NET/SignalR
---
# Стриминг с сервера
Стриминг с сервера работает похоже: клиент запрашивает поток с сервера, обрабатывает сообщения, поступающие от сервера и может прервать передачу при необходимости.
## Добавляем северный стриминг к хабу
Сперва добавим к файлу **LearningHub.cs** ещё один юзинг:
```csharp
using System.Runtime.CompilerServices;
```
Затем добавим такой метод:
```csharp
public async IAsyncEnumerable<string> TriggerStream (
	int jobsCount,
	[EnumeratorCancellation]
	CancellationToken cancellationToken)
{
	for (var i = 0; i < jobsCount; i++)
	{
		cancellationToken.ThrowIfCancellationRequested();
		yield return $"Job {i} executed successfully";
		await Task.Delay(1000, cancellationToken);
	}
}
```
При помощи `cancellationToken`, помеченного атрибутом `EnumeratorCancellation`, можно оборвать поток со стороны клиента.
## Добавляем слушателя серверного потока в клиент JavaScript
Сперва, как обычно, добавим разметку:
```html
<div class="control-group">
	<div>
		<label for="number-of-jobs">NUmber of Jobs</label>
		<input type="text" id="number-of-jobs" name="number-of-jobs" />
	</div>
	<button id="btn-trigger-stream">Trigger Server Stream</button>
</div>
```
Далее добавим обработчик кнопки в файл **wwwroot/js/site.js**:
```js
$('#btn-trigger-stream').click(function () {
	var numberOfJobs = parseInt($('#number-of-jobs').val(), 10);
	connection.stream("TriggerStream", numberOfJobs)
		.subscribe({
			next: (message) => $('#signalr-message-panel')
				.prepend($('<div />').text(message))
		});
});
```
Здесь мы вызываем метод `TriggerStream` и передаём значение для параметра `numberOfJobs`, затем подписываемся на поток и печатаем получаемые сообщения.
## Добавляем слушателя серверного потока в клиент .NET
Сперва добавим новый пункт в наше меню:
```csharp
//...предыдущие пункты...
Console.WriteLine("7 - trigger a server stream");
```
Теперь добавим обработку команды `7`:
```csharp
case "7":
	Console.WriteLine("Please specify the number of jobs to execute");
	var numberOfJobs = int.Parse(Console.ReadLine() ?? "0");
	var cancellationTokenSource = new CancellationTokenSource();
	var stream = hubConnection.StreamAsync<string>("TriggerStream", numberOfJobs, CancellationTokenSource.Token);

	await foreach (var reply in stream)
	{
		Console.WriteLine(reply);
	}
```