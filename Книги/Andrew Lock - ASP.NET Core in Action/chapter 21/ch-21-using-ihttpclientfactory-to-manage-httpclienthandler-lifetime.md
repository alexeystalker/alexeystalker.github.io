---
share: true
tags:
 - NET/HttpClient
---
# Использование IHttpClientFactory для управления жизненным циклом HttpClientHandler
Начнём с того, что класс HttpClient отвечает за оркестровку запросов, но не за само соединение. Вместо этого он  вызывает конвейер `HttpMessageHandler`,  в конце которого находится `HttpClientHandler`, который устанавливает фактическое соединение и отправляет HTTP-запрос, как показано на картинке.
![[Pasted image 20221019210103.png]]
Эта конфигурация напоминает [[ch-3-middleware-pipeline|конвейер промежуточного ПО]], хоть это и *исходящий* конвейер. Каждый обработчик может изменить запрос до того, как последний `HttpClientHandler` выполнит настоящий запрос, а также получает возможность посмотреть ответ после получения.
[[ch-21-calling-http-apis-problem-with-httpclient|Проблемы]] исчерпания сокетов и ротации DNS связаны с удалением `HttpClientHandler` в конце конвейера. По умолчанию удаление `HttpClient` удаляет и весь конвейер обработчиков; `IHttpClientFactory` отделяет жизенный цикл `HttpClient` от базового `HttpClientHandler`.
Такое разделение жизненного цикла позволяет решать проблемы сокетов и DNS двумя способами:
- *путём создания пула доступных обработчиков* — `IHttpClientFactory` поддерживает *активный* обработчик, который используется при создании всех `HttpClient` в течении двух минут, при этом при удалении `HttpClient` базовый обработчик не удаляется, тем самым соединение не закрывается;
- *периодически удаляя обработчики* — каждые две минуты `IHttpClientFactory` создаёт новый активный обработчик, который используется для создания новых `HttpClient`.

Также `IHttpClientFactory` периодически удаляет обработчики с "истекшим сроком действия", если они больще не используются `HttpClient`, обеспечивая использование ограниченного количества подключений[^1].

`IHttpClientFactory` по умолчанию включён в сервисы ASP.NET Core; нужно добавить его в сервисы приложения:
```csharp
public void ConfigureServices(IServiceCollection services)
{
	services.AddHttpClient();
}
```

Теперь мы можем получить объект `IHttpClientFactory` через [[dependency-injection-pattern|внедрение зависимостей]]:
```csharp
[ApiController]
public class ValuesController : ControllerBase
{
	private readonly IHttpClientFactory _factory;
	public ValuesController(IHttpClientFactory factory)
	{
		_factory = factory;
	}
	
	[HttpGet("values")]
	public async Task<string> GetRates()
	{
		HttpClient client = _factory.CreateClient();
		client.BaseAddress = new Uri("https://api.exchangeratesapi.io");
		client.DefaultRequestHeaders
			.Add(HeaderNames.UserAgent, "ExchangeRateViewer");
		
		var response = await client.GetAsync("latest");
		
		response.EnsureSuccessStatusCode();
		return await response.Content.ReadAsStringAsync();
		
	}
}
```
> [!Note]- SocketsHttpHandler и IHttpClientFactory
> Ограничения `HttpClient`, описанные в [[ch-21-calling-http-apis-problem-with-httpclient|предыдущем разделе]], применяются именно к `HttpClientHandler`. `IHttpClientFactory` управляет жизненным циклом и повторным использованием `HttpClientHandler`.
> В .NET Core 2.1 была представлена замена `HttpClientHandler` — `SocketsHttpHandler`, в первую очередь нацеленная на повышение производительности и согласованности между платформами. *Также* `SocketsHttpHandler` можно настроить для использования пула соединений и повторного использования.
> Несмотря на то, что использование `IHttpClientFactory` по прежнему предпочтительно, в сценарии, где нет [[dependency-injection-pattern|внедрения зависимостей]] и где нельзя использовать `IHttpClientFactory`, можно активировать пул соединений `SocketsHttpHandler`, как описано [здесь](https://www.stevejgordon.co.uk/httpclient-connection-pooling-in-dotnet-core)

[^1]: Подробнее об этом [в блоге автора](https://andrewlock.net/exporing-the-code-behind-ihttpclientfactory/)
