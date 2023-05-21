---
share: true
tags:
 - NET/ASPNETCore/testing
 - testing
---
# Тестирование приложения с помощью класса WebApplicationFactory
Предположим, мы хотим протестировать приложение целиком, с полностью настроенным конвейером промежуточного ПО и всеми необходимыми зависимостями в [[di-container|контейнере зависимостей]]. Для этого можно воспользоваться классом `WebApplicationFactory` из пакета Microsoft.AspNetCore.Mvc.Testing.
Рассмотрим пример, в котором мы проверяем, что на запрос `"/ping"` всегда возвращается `"pong"`.
```csharp
public class IntegrationTests: IClassFixture<WebApplicationFactory<Startup>>
{
	private readonly WebApplicationFactory<Startup> _fixture;
	public IntegrationTests(WebApplicationFactory<Startup> fixture)
	{
		_fixture = fixture;
	}
	
	[Fact]
	public async Task PingRequest_ReturnsPong()
	{
		HttpClient = _fixture.CreateClient();
		var response = await client.GetAsync("/ping");
		response.EnsureSuccessStatusCode();
		var content = await response.Content.ReadAsStringAsync();
		Assert.Equal("pong", content);
	}
}
```
Одно из преимуществ использования класса `WebApplicationFactory` состоит в том, что для него требуется меньше ручной настройки.
Также видно, что в отличие от [[ch-23-creating-a-testserver-using-testhost-package|предыдущего раздела]] мы просто тестируем поведение приложения при определенных входных данных, а не проверяем работоспособность определенного класса в контексте фиктивного приложения ASP.NET Core.
Чтобы создать тесты, использующие класс `WebApplicationFactory`, нужно:
1. установить Nuget-пакет Microsoft.AspNetCore.Mvc.Testing в проект;
2. обновить элемент `<Project>` в файле .csproj тестового проекта:
	```xml
	<Project Sdk="Microsoft.NET.Sdk.Web">
	```
	Это необходимо, чтобы класс `WebApplicationFactory` мог найти конфигурационные и статические файлы;
3. унаследовать тестовый класс от `IClassFixture<WebApplicationFactory<T>>`, где `T` - реальный класс в проекте приложения (по соглашению обычно используется класс `Startup`).
	-  `WebApplicationFactory` использует ссылку на `T` для поиска метода `Program.CreateHostBuilder()` для создания `TestServer`;
	-  `IClassFixture<TFixture>` — интерфейс-маркер для xUnit, указывающий на необходимость создать экземпляр `TFixture` и внедрить его в конструктор тестового класса. [Подробнее тут](https://xunit.net/docs/shared-context);
4. принять экземпляр `WebApplicationFactory<T>` в конструкторе тестового класса. Его можно использовать для создания тестового `HttpClient`.