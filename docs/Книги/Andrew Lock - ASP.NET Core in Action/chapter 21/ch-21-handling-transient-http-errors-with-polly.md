---
share: true
tags:
 - NET/HttpClient
---
# Обработка временных ошибок HTTP с помощью библиотеки Polly
[[ch-21-using-ihttpclientfactory-to-manage-httpclienthandler-lifetime|Ранее]] уже упоминалось, что `HttpClient` состоит из "конвейера" обработчиков. Существенное преимущество такого подхода состоит в том, что он позволяет решать сквозные задачи для всех запросов. Например, `IHttpClientFactory` автоматически добавляет обработчик, регистрирующий код состояния и продолжительность каждого исходящего запроса.
Помимо журналирования, ещё одним распространённым требованием является обработка временных ошибок при вызове внешнего API. В случае таких ошибок иногда может быть успешным даже простой повтор запроса, однако необходимость писать для этого код вручную довольно обременительна.
Для решения подобных задач существует библиотека [[library-polly|Polly]]. Для примера добавим простую стратегию повторной отправки запросов. Нам понадобится выполнить следующие шаги:
1. установить пакет NuGet Microsoft.Extensions.Http.Polly;
2. настроить [[ch-21-configuring-named-clients-at-registration-time|именованного]] или [[ch-21-using-typed-clients-to-encapsulate-http-calls|типизированного]] клиента;
3. настроить стратегию:
	```csharp
	public void ConfigureServices(IServiceCollection services)
	{
		services.AddHttpClient<ExchangeRatesClient>()
			.AddTransientHttpErrorPolicy(policy =>
				policy.WaitAndRetryAsync(new[] {
					TimeSpan.FromMilliseconds(200),
					TimeSpan.FromMilliseconds(500),
					TimeSpan.FromSeconds(1)
				})
			);
	}
	```
	
В приведенном примере мы настраиваем обработчик ошибок так, чтобы он, при возникновении ошибки, делал паузу указанной продолжительности, затем повторял запрос, возвращая ошибку только после трёх неуспешных повторов. По умолчанию будет повторяться запрос, который либо:
- возбуждает исключение `HttpRequestException`, указывая на ошибку на уровне протокола;
- возвращает код состояния HTTP 5xx, указывая на ошибку сервера;
- возвращает код состояния HTTP 408, указывая на тайм-аут.

> [!tip] Совет
> О возможностях более тонкой настройки можно прочести в [документации к библиотеке Polly](https://github.com/App-vNext/Polly/wiki/Polly-and-HttpClientFactory)

