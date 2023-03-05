---
share: true
tags:
 - NET/SignalR
 - Azure
---
# Обзор Azure SignalR Service REST API
Преимуществом использования REST API будет возможность отправлять сообщения без использования дополнительных NuGet-пакетов, используя любые HTTP клиенты. Недостатком такого использования будет необходимость написать собственный код, конвертирующий объекты, используемые хабом, в JSON для HTTP запросов. Также потребуется написать свою логику аутентификации.
Попробуем добавить код, отправляющий сообщения при помощи стандартного класса `HttpClient`[^1]. Для этого откроем **HomeController.cs** проекта SignalRServer2.
Сперва добавим юзинги
```csharp
using Newtonsoft.Json;
using System.NetHttp.Headers;
using System.Text;
```
Далее добавим Url
```csharp
private readonly string signalRHubUrl;

public HomeController()
{
	signalRHubUrl = "https://%имя-инстанса-azure%.service.signalr.net/api/v1/hubs/learningHub";
}
```
Далее дополним код [[action-method|метода действия]] `Index()` следующим
```csharp
using var client = new HttpClient();

var payloadMessage = new
{
	Target = "ReceiveMessage",
	Arguments = new[]
	{
		"Client connected to a secondary web application"
	}
};

var request = new HttpRequestMessage(HttpMethod.Post, new UriBuilder(signalRHubUrl).Uri);
request.Headers.Add("Authorization", "Bearer %JWT, который нужно получить у Azure%");
request.Headers.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
request.Content = new StringContent(
	JsonConvert.SerializeObject(payloadMessage),
	Encoding.UTF8,
	"application/json");

var response = await client.SendAsync(request, HttpCompletionOption.ResponseHeadersRead);

if (!response.IsSuccessStatusCode)
	throw new Exception("Failure sending SignalR message.");
```
Теперь, если кто-то откроет домашнюю страницу SignalRServer2 в браузере, все клиенты SignalR, подключённые к нашему хабу при помощи Azure SignalR Service, получат сообщение “Client connected to a secondary web application”.

## Полный список методов Azure SignalR Service REST API[^2]
- `POST /api/v1/hubs/{hub}` - отправить сообщение всем клиентам хаба;
- `POST /api/v1/hubs/{hub}/users/{id}` - отправить сообщение всем клиентам, соответствующим пользователю;
- `POST /api/v1/hubs/{hub}/connections/{connectionId}` - отправить сообщение указаному подключению;
- `GET /api/v1/hubs/{hub}/connections/{connectionId}` - проверить, существует ли подключение с указанным Id;
- `DELETE /api/v1/hubs/{hub}/connections/{connectionId}` - закрыть подключение с указанным Id;
- `POST /api/v1/hubs/{hub}/groups/{group}` - отправить сообщение всем клиентам в указанной группе;
- `GET /api/v1/hubs/{hub}/groups/{group}` - проверить, есть ли подключения в указанной группе;
- `GET /api/v1/hubs/{hub}/users/{user}` - проверить, есть ли подключения, соответствующие указанному пользователю;
- `PUT /api/v1/hubs/{hub}/groups/{group}/connections/{connectionId}` - добавить подключение в группу;
- `DELETE /api/v1/hubs/{hub}/groups/{group}/connections/{connectionId}` - удалить подключение из группы;
- `GET /api/v1/hubs/{hub}/groups/{group}/users/{user}` - проверить, есть ли указанный пользователь в группе;
- `PUT /api/v1/hubs/{hub}/groups/{group}/users/{user}` - добавить пользователя в группу;
- `DELETE /api/v1/hubs/{hub}/groups/{group}/users/{user}` - удалить пользователя из группы;
- `DELETE /api/v1/hubs/{hub}/users/{user}/groups` - удалить пользователя из всех групп.

Как мы видим, здесь есть дополнительные, по сравнению с `HubContext`, методы. Появляется дополнительная сущность “пользователь”, с которой можно ассоциировать подключения, а также добавлять в группы. Чтобы использовать эту фичу, нужно добавить клейм `nameid` к полезной нагрузке вашего [[json-web-token|JWT]]. Это будет идентификатором пользователя, который можно будет использовать в методах API.
## Аутентификация в Azure SignalR Service REST API
Azure SignalR Service REST API использует стандартный JWT в качестве Bearer Token внутри заголовка Authorization. Обзор протоколов OpenId Connect и OAuth дан в [[ch-8-securing-your-signalr-applications|главе 8]]. Об особенностях применения этих протоколов в контексте Azure SignalR Service можно прочитать [здесь](https://learn.microsoft.com/en-us/azure/azure-signalr/signalr-concept-authenticate-oauth).
Две важных вещи, о которых нужно обязательно помнить при использовании JWT с Azure SignalR Service REST API:
1. JWT должен содержать стандартный [[claim|клейм]] `aud`, и его значение должен совпадать с URL, на который отправляется запрос;
2. JWT должен содержать стандартный клейм `exp`, и его значение должно представлять собой время протухания токена в формате unix-time (в оригинале epoch time).

[^1]: дальнейший код упрощён в демострационных целях. Как правильно использовать `HttpClient`, рассказывает Э. Лок [[ch-21-creating-httpclients-with-ihttpclientfactory|в этой главе]]
[^2]: актуальный список методов можно найти [здесь](https://learn.microsoft.com/en-us/azure/azure-signalr/signalr-reference-data-plane-rest-api)