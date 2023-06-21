---
share: true
tags:
 - NET/library
 - NET/testing
---
# WireMock.NET
WireMock.NET — это .NET-версия [WireMock](https://wiremock.org/), библиотеки для заглушек и имитации HTTP-сервисов. С WireMock.NET вы можете определить ожидаемые ответы для конкретных запросов, а библиотека будет перехватывать и управлять этими запросами для вас. Это позволяет легко тестировать код, выполняющий HTTP-запросы, не полагаясь на фактическую доступность внешнего сервиса и не взламывая HttpClient.

После установки nuget-пакета вы можете начать использовать WireMock.NET, создав экземпляр класса `WireMockServer` и настроив его на желаемое поведение. Это можно легко сделать, вызвав метод `WireMockServer.Start` или `WireMockServer.StartWithAdminInterface`:
```csharp
using var wireMock =  WireMockServer.Start(port: 1080, ssl: false);
```
Теперь можно начать определять моки для внешних HTTP-запросов, в том числе, используя Fluent API:
```csharp
wireMock
	.Given(Request.Create()
		.WithPath("/foo")
		.UsingGet())
	.RespondWith(Response.Create()
		.WithStatusCode(200)
		.WithHeader("Content-Type", "application/json; charset=utf-8")
	    .WithBodyAsJson(new { msg = "Hello world!" }));
```
В тестах легко написать проверку результатов HTTP-запроса:
```csharp
[Fact]
public async Task sample_test()
{
	var response = await new HttpClient().GetAsync($"{_wireMock.Urls[0]}/foo");

	Assert.Equal(HttpStatusCode.OK, response.StatusCode);
	Assert.Equal("""{"msg":"Hello world!"}""", await response.Content.ReadAsStringAsync());
}
```
> [! Note] Замечание
> В реальном приложении, конечно, тестироваться будут не собственно HTTP-ответы, а код, обрабатывающий эти ответы.

WireMock.NET предлагает множество функций, помимо базовых заглушек и имитаций HTTP-запросов:
1. Прокси-запросы к реальному сервису (https://github.com/WireMock-Net/WireMock.Net/wiki/Proxying) и захват ответов в виде маппинга. Вы можете использовать их в качестве основы для своих заглушек, устраняя необходимость в ручном определении ответов.
2. Чтение маппингов и определение заглушек из статических файлов (https://github.com/WireMock-Net/WireMock.Net/wiki/Mapping#StaticMappings), вместо того, чтобы определять их программно. Это может быть полезно для совместного использования заглушек в разных тестах или проектах.
3. Создание динамических шаблонов ответов (https://github.com/WireMock-Net/WireMock.Net/wiki/Response-Templating), которые включают данные из запроса. Это позволяет создавать ответы, которые различаются в зависимости от данных запроса, что может быть полезно для тестирования пограничных случаев или моделирования поведения реального сервиса.
4. Моделирование поведения сервиса с помощью сценариев и состояний (https://github.com/WireMock-Net/WireMock.Net/wiki/Scenarios-and-States). Вы можете легко имитировать различные состояния сервиса и переключаться между ними. Это может быть полезно для проверки того, как ваш код обрабатывает различные типы сбоев или ответов от сервиса.

## Ссылки
[Вики](https://github.com/WireMock-Net/WireMock.Net/wiki)
https://cezarypiatek.github.io/post/mocking-outgoing-http-requests-p1/
https://t.me/NetDeveloperDiary/1903