---
share: true
tags:
 - NET/HttpClient
---
# Использование типизированных клиентов для инкапсуляции HTTP-вызовов
Существует распространённый шаблон взаимодействия с API — инкапсуляция механизма взаимодействия в отдельный сервис. У `IHttpClientFactory` есть основательная поддержка такого шаблона — *типизированные клиенты*.
*Типизированный клиент* — это класс, принимающий `HttpClient` в конструкторе и предоставляющий понятный интерфейс для потребителей, инкапсулируя детали взаимодействия с API — HTTP-методы, заголовки, пути методов и т.п.
Вот для примера типизированный клиент для API курсов обмена валют:
```csharp
public class ExchangeRatesClient
{
	private readonly HttpClient _client;
	public ExchangeRatesClient(HttpClient client)
	{
		_client = client;
	}
	
	public async Task<string> GetLatestRates()
	{
		var response = await _client.GetAsync("latest");
		response.EnsureSuccessStatusCode();
		
		return await response.Content.ReadAsString();
	}
}
```
Затем внедрим его в наш контроллер:
```csharp
[ApiController]
public class ValuesController: ControllerBase
{
	private readonly ExchangeRatesClient _ratesClient;
	public ValuesController(ExchangeRatesClient ratesClient)
	{
		_ratesClient = ratesClient;
	}
	
	[HttpGet("values")]
	public async Task<string> GetRates()
	{
		return await _ratesClient.GetLatestRates();
	}
}
```
Но как тут задействован `IHttpClientFactory`? `IHttpClientFactory` создаёт `HttpClient`, настраивает его и внедряет в экземпляр типизированного клиента.
```csharp
public void ConfigureServices(IServiceCollection services)
{
	services.AddHttpClient<ExchangeRatesClient>((HttpClient client) =>
	{
		client.BaseAddress = new Uri("https://api.exchangeratesapi.io");
		client.DefaultRequestHeaders
			.Add(HeaderNames.UserAgent, "ExchangeRateViewer");
	})
	.ConfigureHttpClient((HttpClient client) => {});//дополнительно
}
```
> [!tip] Совет
> Типизированного клиента можно рассматривать как оболочку для [[ch-21-configuring-named-clients-at-registration-time|именованного клиента]]. Часто это оправданный подход, так как он объединяет логику взаимодействия с удалённой службой в одном месте, а также избегает "волшебных строк", которые используются с именованными клиентами.

Ещё один часто используемый подход — помимо реализации зарегистрировать интерфейс. Этот подход значительно упрощает тестирование.
```csharp
services.AddHttpClient<IExchangeRatesClient, ExchangeRatesClient>()
```

Ещё один вариант — перенести конфигурацию из `ConfigureServices` в конструктор типизированного клиента.
```csharp
public class ExchangeRatesClient
{
	private readonly HttpClient _client;
	public ExchangeRatesClient(HttpClient client)
	{
		_client = client;
		_client.BaseAddress = new Uri("https://api.exchangeratesapi.io");
		_client.DefaultRequestHeaders
			.Add(HeaderNames.UserAgent, "ExchangeRateViewer");
	}
	
	public async Task<string> GetLatestRates()
	{
		var response = await _client.GetAsync("latest");
		response.EnsureSuccessStatusCode();
		
		return await response.Content.ReadAsString();
	}
}
```

