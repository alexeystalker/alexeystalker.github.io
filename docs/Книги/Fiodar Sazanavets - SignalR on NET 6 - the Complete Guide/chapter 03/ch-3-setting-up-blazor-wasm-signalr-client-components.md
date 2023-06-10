---
share: true
tags:
 - NET/SignalR
 - NET/BlazorWasm
---
# Настроим компоненты SignalR
Сперва добавим NuGet-пакет клиента SignalR:
```bash
dotnet add package Microsoft.AspNetCore.SignalR.Client
```
Теперь добавим логику клиента. Располагаться она будет в файле **Client.razor** внутри папки **Pages**:
```razor
@page "/client"
@using Microsoft.AspNetCore.SignalR.Client
@inject NavigationManager NavigationManager
@implements IAsyncDisposable

<h1>Blazor WebAssembly Client</h1>

<div class="row" style="padding-top: 50px;">
	<div class="col-md-4">
		<div class="control-group">
			<div>
				<label for="broadcastMsg">Message</label>
				<input @bind="message" type="text" id="broadcastMsg" name="broadcastMsg" />
			</div>
			 <button @onclick="BroadcastMessage" disabled="@(!IsConnected)">Broadcast</button>
		 </div>
	 </div>
	 
	 <div class="col-md-7">
	 	<p>SignalR Messages:</p>
		<pre id="signalr-message-panel">
			@foreach (var message in messages)
			{
				@message<br />
			}
		</pre>
	</div>
</div>
```
Здесь мы начали с того, что указали на дефолтный путь к странице в директиве `@page`. Далее добавили ссылки на внешние пространства имён через директиву `@using`. Затем добавили зависимость от `NavigationManager`через директиву `@inject` — он понадобится позже для получения URL к хабу SignalR. Наконец, использовали директиву `@implements` для указания того факта, что наш компонент реализует интерфейс `IAsyncDisposable`.
Далее, внутри HTML разметки мы добавили несколько ключевых слов, начинающихся на `@`. Так, `@bind` связывает контент элемента `input` с переменной `message`. Указание о том, что нажатие на кнопку вызывает выполнение метода `BroadcastMessage` выражено через `@onclick`. И, наконец, логика внутри элемента `<pre>` заполняет элемент содержимым коллекции `messages`.
Но нам ещё надо добавить некоторое количество кода. Добавим его в конец файла:
```csharp
@code {
	private HubConnection hubConnection;
	private List<string> messages = new List<string>();
	private string? message;
	
	protected override async Task OnInitializedAsync()
	{
		hubConnection = new HubConnectionBuilder()
			.WithUrl(NavigationManager.ToAbsoluteUri("/learningHub"))
			.Build();
		
		hubConnection.On<string>("ReceiveMessage", (message) =>
		{
			messages.Add(message);
			StateHasChanged();
		});
		await hubConnection.StartAsync();
	}
	
	private async Task BroadcastMessage() =>
		await hubConnection.SendAsync("BroadcastMessage", message);
	
	public bool IsConnected =>
		hubConnection?.State == hubConnectionState.Connected;
	
	public async ValueTask DisposeAsync()
	{
		await hubConnection.DisposeAsync();
	}
}
```
Здесь мы уже используем чистый C#, однако в основном код очень схож с тем, что мы [[ch-3-adding-js-signalr-client-logic|видели]] в случае для JavaScript. Сперва строим объект `hubConnection` при помощи `HubConnectionBuilder`, затем регистрируем обработчик события `ReceiveMessage`, которое может вызвать сервер, в котором добавляем полученное сообщение в коллекцию полученных сообщений, и вызываем специфичную для Blazor функцию `StateHasChanged`, которая запускает перевыполнение кода, определённого в разметке. При этом панель с полученными сообщениями всегда отображает все сообщения, включая и полученное только что. Метод `BroadcastMessage` отправляет введённое сообщение на сервер. Свойство `IsConnected` отвечает за то, что соответствующие контролы появляются на странице только тогда, когда соединение действительно установлено. Для того, чтобы определить, что соединение больше не нужно, вызываем `hubConnection.DisposeAsync()`.