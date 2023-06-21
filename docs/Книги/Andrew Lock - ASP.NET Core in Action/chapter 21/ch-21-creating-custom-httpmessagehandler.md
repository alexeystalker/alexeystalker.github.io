---
aliases:
 - HttpMessageHandler
share: true
tags:
 - NET/HttpClient
---
# Создание специального обработчика HttpMessageHandler
Для большинства сторонних API при их вызове требуется некая форма аутентификации. Очень частым вариантом является добавление API-ключа (*ApiKey*) к исходящему запросу. Создадим собственный обработчик `HttpMessageHandler` для выполнения этой задачи.
>[!note]- Примечание
>Более сложные API-интерфейсы могут использовать [[json-web-token|JWT]], полученные от поставщика идентификационной информации. В этом случае можно рассмотреть возможность использования [библиотеки IdentityModel](https://identitymodel.readthedocs.io), обеспечивающей точки интеграции для ASP.NET Core Identity и `IHttpClientFactory`.

Наш обработчик будет частью обработчиков `HttpClient`.
![[Pasted image 20221022131106.png]]
Для создания собственного обработчика нужно выполнить следующее.
1. Создаём обработчик, наследуя от базового класса `DelegatingHandler`;
2. Переопределяем метод `SendAsync()` для выполнения нужной задачи. Вызываем `base.SendAsync()` для выполнения оставшейся части конвейера;
3. Регистрируем обработчик в [[di-container|DI контейнере]] с жизненным циклом Singleton (если не требуется состояние) или Transient;
4. Добавляем обработчик к одному или нескольким [[ch-21-configuring-named-clients-at-registration-time|именованным]] или [[ch-21-using-typed-clients-to-encapsulate-http-calls|типизированным]] клиентам, вызвав метод `AddHttpMessageHandler<T>()`. Порядок, в котором регистрируются обработчики, определяет порядок их добавления в конвейер. При необходимости один и тот же тип обработчика может быть добавлен несколько раз, а также для нескольких клиентов.

Вот пример специального обработчика, добавляющего заголовок к каждому запросу.
```csharp
public class ApiKeyMessageHandler: DelegatingHandler
{
	private readonly ExchangeRateApiSettings _settings;
	public ApiKeyMessageHandler(IOptions<ExchangeRateApiSettings> settings)
	{
		_settings = settings.Value;
	}
	
	protected override async Task<HttpResponseMessage> SendAsync(
		HttpRequestMessage request,
		CancellationToken cancellationToken)
	{
		request.Headers.Add("X-API-KEY", _settings.ApiKey);
		var response = await base.SendAsync(request, cancellationToken);
		//Если необходимо, ответ можно проверить или изменить
		return response;
	}
}
```
Теперь регистрируем обработчик и добавляем его к именованному или типизированному клиенту.
```csharp
public void ConfigureServices(IServiceCollection services)
{
	services.AddTransient<ApiKeyMessageHandler>();
	
	services.AddHttpClient<ExchangeRatesClient>()
		.AddHttpMessageHandler<ApiKeyMessageHandler>()
		.AddTransientHttpErrorPolicy(policy =>
			policy.WaitAndRetryAsync(new[] {
				TimeSpan.FromMilliseconds(200),
				TimeSpan.FromMilliseconds(500),
				TimeSpan.FromSeconds(1)
			})
		);
}
```

