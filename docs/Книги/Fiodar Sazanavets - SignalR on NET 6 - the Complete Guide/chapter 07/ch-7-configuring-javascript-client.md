---
share: true
tags:
 - NET/SignalR
 - JavaScript
---
# Настраиваем клиент JavaScript
Откроем файл **wwwroot/js/site.js** и найдем место, где инициализируется объект `connection`. Заменим реализацию на следующее:
```js
const connection = new signalR.HubConnectionBuilder()
	.withUrl("/learningHub", {
		transport: signalR.HttpTransportType.WebSockets | signalR.HttpTransportType.LongPolling,
		headers: { "Key": "value" },
		accessTokenFactory: null,
		logMessageContent: true,
		skipNegotiation: false,
		withCredentials: true,
		timeout: 10000
	})
	.configureLogging(signalR.LogLevel.Information)
	.build();
```
вот какие здесь есть настройки[^1]:
- `transport` — какие транспорты поддерживает клиент. По умолчанию доступны все три транспорта, с приоритетом WebSockets;
- `headers` — заголовки, которые нужно добавить к запросу на подключение;
	> [!warning] Внимание (от меня)
	> В документации также сказано, что, цитирую “Note, setting headers in the browser will not work for WebSockets or the ServerSentEvents stream.” Видимо, в браузере передача заголовков не сработает
- `accessTokenFactory` — фабрика токенов аутентификации[^2];
- `logMessageContent` — нужно ли добавлять содержимое сообщений в логи;
- `skipNegotiation` — если используется WebSocket в качестве транспорта, эта опция позволяет пропустить предварительный этап. Иначе не используется;
- `withCredentials` — если задано, данные входа будут отправлены при Кросс-доменном запросе(CORS)[^3]
- `timeout` — таймаут для запросов HTTP. Не относится к LongPolling, SSE или WebSockets.

`configureLogging` — метод для конфигурации логирования. Можно передать уровень логирования в виде строки или константы, или передать кастомный `ILogger`.

После того, как объект `connection` создан, можно добавить следующие строки:
```js
connection.serverTimeoutInMilliseconds = 30000;
connection.keepAliveIntervalInMilliseconds = 15000;
```
Здесь:
- `serverTimeoutInMilliseconds` — таймаут ожидания сообщения от сервера. При превышении соединение считается разорванным и вызывается обработчик `onclose`;
- `keepAliveIntervalInMilliseconds` — интервал отправки пинг-сообщений на сервер. По умолчанию 15 секунд.

[^1]: - [Документация HubConnectionBuilder](https://learn.microsoft.com/en-us/javascript/api/@microsoft/signalr/hubconnectionbuilder?view=signalr-js-latest)
	- [Документация с описанием настроек](https://learn.microsoft.com/en-us/javascript/api/@microsoft/signalr/ihttpconnectionoptions?view=signalr-js-latest)

[^2]:О аутентификации на клиентах [[ch-8-authentication-on-clients|тут]].
[^3]:[[ch-18-calling-web-apis-from-other-domains-using-cors|См. про CORS в книге Э. Лока]]