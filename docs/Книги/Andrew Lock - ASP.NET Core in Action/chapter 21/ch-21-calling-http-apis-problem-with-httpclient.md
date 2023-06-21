---
share: true
tags:
 - NET/HttpClient
---
# Вызов API для протокола HTTP: проблема с классом HttpClient
Рассмотрим, как использовать класс HttpClient для вызова удалённых API, а также как именно могут возникнуть две распространённые ошибки — исчерпание сокетов и проблемы ротации DNS. В [[ch-21-creating-httpclients-with-ihttpclientfactory|следующем разделе]] мы увидим, как избежать этих проблем при помощи `IHttpClientFactory`.

В .NET мы используем класс `HttpClient`, для выполнения HTTP-вызовов, предоставления нужных заголовков и тела запроса, а также чтения заголовков ответов и получаемых данных.
Рассмотрим пример, как не нужно использовать класс `HttpClient`.
> [!warning] Внимание!
> Использование кода следующего примера может привести к нестабильности вашего приложения!

```csharp
[ApiController]
public class ValuesController : ControllerBase
{
	[HttpGet("values")]
	public async Task<string> GetRates()
	{
		using (HttpClient client = new HttpClient())
		{
			client.BaseAddress = new Uri("https://api.exchangeratesapi.io");
			var response = await client.GetAsync("latest");
			response.EnsureSuccessStatusCode();
			return await response.Content.ReadAsStringAsync();
		}
	}
}
```
Казалось бы, так как `HttpClient` реализует интерфейс `IDisposable`, его использовать обычным для реализации этого интерфейса способом. Однако, в данном случае так поступать *не следует*! Подобный код может привести к проблеме, называемой *исчерпание сокетов (socket exhaustion)*. Проблема в том, когда мы удаляем (dispose) объект `HttpClient`, он *не закрывает сокет немедленно*. Сперва сокет должен перейти в состояние, называемое `TIME_WAIT`, и пребывает в этом состоянии определенное время (для Windows это время составляет **240 с**), прежде чем полностью закрыться. При этом его невозможно использовать для других запросов.
Если приложение делает много запросов - это быстро приведёт к исчерпанию сокетов.
> [!Note]- Совет
> Посмотреть состояние активных портов или сокетов можно при помощи команды `netstat`. В Windows обязательно нужно использовать `netstat -n`, чтобы пропустить разрешение DNS.

Поэтому, вместо того, чтобы избавляться от `HttpClient`, рекомендовалось (до того, как в .NET Core 2.1 появился `IHttpClientFactory`) использовать один экземпляр `HttpClient`:
```csharp

[ApiController]
public class ValuesController : ControllerBase
{
	private static readonly HttpClient _client = new HttpClient
	{
		BaseAddress = new Uri("https://api.exchangeratesapi.io")
	}
	
	[HttpGet("values")]
	public async Task<string> GetRates()
	{
		var response = await _client.GetAsync("latest");
		response.EnsureSuccessStatusCode();
		return await response.Content.ReadAsStringAsync();
	}
}
```
Это решает проблему исчерпания сокетов, поскольку объект `HttpClient` не удаляется, а переиспользуется для нескольких запросов.
Однако, это приводит нас к другой проблеме — такой подход не обнаруживает изменения в DNS. Имя хоста резолвится один раз при создании экземпляра `HttpClient`, и впоследствии не меняется. Однако, если DNS-запись сервиса, который мы используем, изменилась за время работы нашего приложения, экземпляр клиента не узнает об этом, и запросы будут уходить на старый адрес.
