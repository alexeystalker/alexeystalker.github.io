---
share: true
tags:
 - NET/ASPNETCore/testing
 - testing
---
# Замена зависимостей в классе WebApplicationFactory
При [[ch-23-testing-your-application-with-webapplicationfactory|использовании]] `WebApplicationFactory` приложение запускается в памяти, но с использованием тех же настроек, секретов или API-ключей, что и при использовании команды `dotnet run`.
С одной стороны, это позволяет правильно протестировать запуск приложения. С другой — означает, что тесты будут использовать те же сервисы и соединения с БД, что и при локальном запуске. При этом часто возникает желание заменить их "тестовыми" версиями сервисов.
Например, у нас есть сервис `CurrencyConverter`, который использует вызов стороннего API. Для тестов мы бы хотели заменить его на, скажем, `StubCurrencyConverter`.
Для этого прежде всего убедимся, что у конвертера выделен интерфейс, и именно он (а не реализация) используется в приложении[^1], а также создадим тестовую реализацию.
Затем настроим `WebApplicationFactory` при помощи предоставляемого метода `WithWebHostBuilder`. Вот код примера.
```csharp
public class IntegrationTests: IClassFixture<WebApplicationFactory<Startup>>
{
	private readonly WebApplicationFactory<Startup> _fixture;
	public IntegrationTests(WebApplicationFactory<Startup> fixture)
	{
		_fixture = fixture;
	}
	
	[Fact]
	public async Task ConvertReturnsExpectedValue()
	{
		var customFactory = _fixture.WithWebHostBuilder((IWebHostBuilder builder) =>
		{
			hostBuilder.ConfigureTestServices(services =>
			{
				services.RemoveAll<ICurrencyConverter>();
				services.AddSingleton<ICurrencyConverter, StubCurrencyConverter>();
			});
		});
		
		var client = customFactory.CreateClient();
		var response = await client.GetAsync("/api/currency");
		response.EnsureSuccessStatusCode();
		var content = await response.Content.ReadAsStringAsync();
		
		Assert.Equal("3", content);
	}
}
```
Отметим следющее:
- метод `WithWebHostBuilder()` возвращает *новый* экземпляр класса `WebApplicationFactory` с новой конфигурацией, не меняя `_fixture`;
- метод `ConfigureTestServices()` вызывается после метода `ConfigureServices()` реального приложения, что позволяет заменить ранее зарегистрированные сервисы, а также использовать его для переопределения значений конфигурации.

[^1]: в книге кратко описано, как выделять интерфейс и регистрировать его в DI, но я опущу это — думаю, все и так хорошо знают, как это делается.