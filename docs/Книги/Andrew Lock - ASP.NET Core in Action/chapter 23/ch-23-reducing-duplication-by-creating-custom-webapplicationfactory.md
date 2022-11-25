---
share: true
tags:
 - NET/ASPNETCore/testing
 - testing
---
# Уменьшение дублирования кода за счёт создания специального класса WebApplicationFactory
Рассмотрим, как вынести [[ch-23-replacing-dependencies-in-webapplicationfactory|логику переопределения зависимостей]] в специальный класс для повторного использования.
```csharp
public class CustomWebApplicationFactory: WebApplicationFactory<Startup>
{
	protected override void ConfigureWebHost(IWebHostBuilder builder)
	{
		builder.ConfigureTestServices(services =>
		{
			services.RemoveAll<ICurrencyConverter>();
			services.AddSingleton
				<ICurrencyConverter, StubCurrencyConverter>();
		});
	}
}
```
В этом примере мы переопределяем метод `ConfigureWebHost` для настройки тестовых сервисов[^1].
Вот как применить полученную фабрику.
```csharp
public class IntegrationTests: IClassFixture<CustomWebApplicationFactory>
{
	private readonly CustomWebApplicationFactory _fixture;
	public IntegrationTests(CustomWebApplicationFactory fixture)
	{
		_fixture = fixture;
	}
	
	[Fact]
	public async Task ConvertReturnsExpectedValue()
	{
		var client = _fixture.CreateClient();
		var response = await client.GetAsync("/api/currency");
		response.EnsureSuccessStatusCode();
		var content = await response.Content.ReadAsStringAsync();
		
		Assert.Equal("3", content);
	}
}
```
При этом метод `WithWebHostBuilder()` остается доступным, что позволяет использовать его для точечных переопределений в конкретном тесте.

[^1]: подробнее об этом, в том числе о других методах, доступных для переопределения, см. [здесь](https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests?view=aspnetcore-5.0)