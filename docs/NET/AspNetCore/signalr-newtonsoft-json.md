---
share: true
tags:
 - NET/SignalR
 - NET/NewtonsoftJson
---
# Используем Newtonsoft.Json в SignalR
SignalR Core по умолчанию использует System.Text.Json для сериализации. Однако иногда требуется использовать наш любимый Newtonsoft.Json (aka Json.Net). Почему? Ну, например, если вы внедряете SignalR в существующий проект, и в вашем WebApi (так исторически сложилось) используется Newtonsoft.Json. А того хуже, если он используется с кастомными настройками, и вы хотите сохранить единообразие ваших json-ов.
Для того, чтобы использовать Newtonsoft.Json, нужно:
1. Подключить пакет [Microsoft.AspNetCore.SignalR.Protocols.NewtonsoftJson](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Protocols.NewtonsoftJson) на сервере (если планируется использование шарпового клиента SignalR — то и в проект клиента тоже);
2. вызвать метод `AddNewtonsoftJsonProtocol()` в конфигурации после `AddSignalR`, вот так:
	```csharp
	builder.Services.AddSignalR().AddNewtonsoftJsonProtocol();
	```
3. В шарповом клиенте (если есть) вызываем тот же метод в `HubConnectionBuilder`:
	```csharp
	var hubConnection = new HubConnectionBuilder()
		.AddNewtonsoftJsonProtocol()
		.Build();
	```
	
При этом `AddNewtonsoftJsonProtocol()` принимает лямбду типа `Action<NewtonsoftJsonHubProtocolOptions>`, у которой есть свойство `PayloadSerializerSettings` типа `JsonSerializerSettings`, то есть того же типа, что и при конфигурации сериализатора для Api. Другими словами, при конфигурации SingalR можно использовать тот же самый конфигурационный код, что и при конфигурации Api, примерно так:
```csharp
builder.Services.AddSignalR().AddNewtonsoftJsonProtocol(options => 
{
	//Вынесем код, конфигурирующий сериализацию, в отдельный метод, и применим здесь:
	ConfigurationHelper
		.ConfigureDefaultSerializerSettings(options.PayloadSerializerSettings);
});
```

## Ссылки
https://towardsdev.com/using-newtonsoft-json-in-asp-net-core-and-signalr-55b0fa4645aa
