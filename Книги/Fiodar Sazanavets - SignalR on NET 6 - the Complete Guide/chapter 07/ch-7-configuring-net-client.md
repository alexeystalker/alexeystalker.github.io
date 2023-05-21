---
share: true
tags:
 - NET/SignalR
---
# Настраиваем клиент .NET
Сперва добавим юзинги в файл **Program.cs** нашего проекта DotnetClient:
```csharp
using Microsoft.AspNetCore.Connections;
using Microsoft.AspNetCore.Http.Connections;
using Microsoft.AspNetCore.SignalR.Client;
using Microsoft.Extensions.Logging;
```

Теперь заменим инициализацию объекта `hubConnection` следующим:
```csharp
var hubConnection = new HubConnectionBuilder()
	.WithUrl(url,
		HttpTransportType.WebSockets,
		options => {
			options.AccessTokenProvider = null;
			options.HttpMessageHandlerFactory = null;
			options.Headers["CustomData"] = "Value";
			options.SkipNegotiation = true;
			options.ApplicationMaxBufferSize = 1_000_000;
			options.ClientCertificates = new System.Security.Cryptography
				.X509Certificates.X509CertificatesCollection();
			options.CloseTimeout = TimeSpan.FromSeconds(5);
			options.Cookies = new System.Net.CookieContainer();
			options.DefaultTransferFormat = TransferFormat.Text;
			options.Credentials = null;
			options.Proxy = null;
			options.UseDefaultCredentials = true;
			options.TransportMaxBufferSize = 1_000_000;
			options.WebSocketConfiguration = null;
			options.WebSocketFactory = null;
		})
	.ConfigureLogging(logging => {
		logging.SetMinimumLevel(logLevel.Information);
		logging.AddConsole();
	})
	.Build();
```
Здесь мы использовали один из методов расширения `WithUrl()` с возможностью указания используемого транспорта в отдельном параметре. Здесь мы указали только `WebSockets`, но здесь также, как и в настройках можно объединять разные варианты при помощи побитового ИЛИ (так как enum `HttpTransportType` имеет атрибут `[Flags]`). Все возможные методы расширения указаны [здесь](https://learn.microsoft.com/ru-ru/dotnet/api/microsoft.aspnetcore.signalr.client.hubconnectionbuilder?view=aspnetcore-6.0#extension-methods).
Кратко упомянем все показанные здесь настройки. [Документация по этим настройкам](https://learn.microsoft.com/ru-ru/dotnet/api/microsoft.aspnetcore.http.connections.client.httpconnectionoptions?view=aspnetcore-6.0). Если не указано иное, под "запросом" подразумевается запрос на подключение.
- `AccessTokenProvider` - задаёт провайдер для токена доступа[^2];
- `HttpMessageHandlerFactory` - задаёт делегат для создания обёртки или замены используемому `HttpMessageHandler`[^1]
- `Headers` - заголовки, которые будут переданы с запросом;
- `SkipNegotiation` - при использовании WebSocket в качестве транспорта этап предварительного соединения может быть пропущен. Поведение регулируется этой настройкой;
- `ApplicationMaxBufferSize` - размер буфера данных, отправляемых приложением; после заполнения буфера будет активирована логика замедления (backpressure);
- `ClientCertificates` - коллекция сертификатов, которые могут быть отправлены с запросом для аутентификации;
- `CloseTimeout` - таймаут ожидания ответа сервера после отправки клиентом запроса на закрытие соединения. После истечения таймаута соединение закрывается принудительно;
- `Cookies` - коллекция кук, отправляемых с запросом;
- `DefaultTransferFormat` - формат передачи данных, используемый по умолчанию;
- `Credentials` - аутентификационные данные (credentials), добавляемые к запросу;
- `Proxy` - позволяет настроить прокси для запросов;
- `UseDefaultCredentials` - позволяет использовать аутентификационные данные по умолчанию, например, текущего пользователя Active Directory;
- `WebSocketConfiguration` - позволяет задать кастомные настройки соединения WebSocket;
- `WebSocketFactory` - задаёт делегат для обёртки или замены стандартного объекта `WebSocket`;

Метод `CondigureLogging` позволяет сконфигурировать логирование стандартными для .NET Core методами.

Также у объекта `hubConnection` есть динамически изменяемые свойства. Вот они:
```csharp
hubConnection.HandshakeTimeout = TimeSpan.FromSeconds(15);
hubConnection.ServerTimeout = TimeSpan.FromSeconds(30);
hubConnection.KeepAliveInterval = TimeSpan.FromSeconds(10);
```
- `HandshakeTimeout` - таймаут ожидания ответа сервера при первоначальном обмене (handshake);
- `ServerTimeout` - таймаут ожидания сообщения от сервера (включая пинги);
- `KeepAliveInterval` - интервал между пингами, отправляемыми на сервер.

> [!Note]- От меня
> Почему-то автор не упомянул здесь о методе расширения `WithAutomaticReconnect()`, (точнее, нескольких перегруженных методах) который позволяет задать политику автоматического переподключения при потере соединения. 

[^1]: О создании кастомного `HttpMessageHandler` можно прочитать в [[ch-21-creating-custom-httpmessagehandler|книге Э. Лока]]
[^2]:О аутентификации на клиентах [[ch-8-authentication-on-clients|тут]]