---
share: true
tags:
 - NET/HttpClient
---
# Настройка именованных клиентов во время регистрации
Представленный в [[ch-21-using-ihttpclientfactory-to-manage-httpclienthandler-lifetime|предыдущем]] разделе код выглядит довольно запутанным, прежде всего потому, что `HttpClient` даже с использованием `IHttpClientFactory`, должен быть настроен перед использованием. Если мы хотим использовать `HttpClient` в разных местах — он должен быть настроен каждый раз перед использованием.
`IHttpClientFactory` позволяет решить эту проблему, централизованно настраивая *именованных клиентов*. У такого клиента есть строковое имя и функция конфигурации, запускающаяся при запросе экземпляра клиента с этим именем.
Зарегистрируем именованного клиента `"rates"`.
```csharp
public void ConfigureServices(IServiceCollection services)
{
	services.AddHttpClient("rates", (HttpClient client) =>
	{
		client.BaseAddress = new Uri("https://api.exchangeratesapi.io");
		client.DefaultRequestHeaders
			.Add(HeaderNames.UserAgent, "ExchangeRateViewer");
	})
	.ConfigureHttpClient((HttpClient client) => {})//дополнительно
	.ConfigureHttpClient(
		(IServiceProvider provider, HttpClient client) => {});
}
```
Тут мы регистрируем клиента, также можно добавить дополнительные методы доконфигурации. Существует перегруженный метод, с доступом к контейнеру [[dependency-injection-pattern|внедрения зависимостей]].
Теперь наш контроллер из предыдущего раздела выглядит так:
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
		var client = _factory.CreateClient("rates");
		
		var response = await client.GetAsync("latest");
		
		response.EnsureSuccessStatusCode();
		return await response.Content.ReadAsStringAsync();
		
	}
}
```
> [!note] Примечание
> Даже при наличии сконфигурированных клиентов, можно продолжать создавать несконфигурированные клиенты, используя `CreateClient()` без имени. Также несконфигурированный клиент будет создан, если передать неизвестное имя. Важно понимать, что строки сравниваются целиком; `"rates"` и `"Rates"` — разные имена.

