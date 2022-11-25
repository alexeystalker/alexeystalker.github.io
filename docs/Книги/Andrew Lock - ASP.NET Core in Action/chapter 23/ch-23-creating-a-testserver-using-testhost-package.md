---
share: true
tags:
 - NET/ASPNETCore/testing
 - testing
---
# Создание TestServer с помощью пакета TestHost
Предположим, что мы хотим написать несколько интеграционных тестов для класса `StatusMiddleware` из [[ch-23-unit-testing-custom-middleware|этого раздела]]. Для этого воспользуемся пакетом TestHost. Добавим его Nuget-пакет Microsoft.AspNetCore.TestHost в наше приложение.
В типичном приложении ASP.NET Core мы создаем `HostBuilder`, настраиваем веб-сервер, определяем конфигурацию приложения, его сервисы и контейнер промежуточного ПО, затем вызываем метод `Build()` для создания экземпляра `IHost`, который может быть запущен и который будет слушать запросы по определенному URL-адресу и порту.
Пакет Test Host использует тот же `HostBuilder` для определения тестового приложения, но при этом создает объект `IHost`, который использует объекты запроса в памяти.
Он даже предоставляет объект `HttpClient`, с помощью которого можно отправлять запросы в тестовое приложение.
Рассмотрим пример, как использовать TestHost для тестирования класса `StatusMiddleware`.
```csharp
public class StatusMiddlewareTests
{
	[Fact]
	public async Task StatusMiddlewareReturnsPong()
	{
		var hostBuilder = new HostBuilder()
			.ConfigureWebHost(webHost =>
			{
				webHost.Configure(app => app.UseMiddleware<StatusMiddleware>());
				webhost.UseTestServer();
			});
		IHost host = await hostBuilder.StartAsync();
		HttpClient client = host.GetTestClient();
		
		var response = await client.GetAsync("/ping");
		
		response.EnsureSuccessStatusCode();
		var content = await response.Content.ReadAsStringAsync();
		Assert.Equal("pong", content);
	}
}
```