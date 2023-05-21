---
share: true
tags:
 - NET/ASPNETCore/testing
 - testing
---
# Модульное тестирование специального промежуточного ПО
Ранее мы [[ch-19-building-custom-middleware-component|рассмотрели]], как создать собственное промежуточное ПО. Теперь же мы напишем модульные тесты для проверки работоспособности такого ПО.
Вот наш класс, который мы хотим протестировать:
```csharp
public class StatusMiddleware
{
	private readonly RequestDelegate _next;
	public StatusMiddleware(RequestDelegate next)
	{
		_next = next;
	}
	
	public async Task Invoke(HttpContext context)
	{
		if(context.Request.Path.StartsWithSegments("/ping"))
		{
			context.Response.ContentType = "text/plain";
			await context.Response,WriteASync("pong");
			return;
		}
		await _next(context);
	}
}
```
Сейчас мы протестируем два простых случая:
- когда запрос выполнен с путём `"/ping"`;
- когда запрос выполнен с другим путём.

Сначала протестируем случай, когда путь запроса не начинается с `"/ping"`:
```csharp
[Fact]
public async Task ForNonMatchingRequest_CallsNextDelegate()
{
	var context = new DefaultHttpContext();
	context.Request.Path = "/somethingelse";
	var wasExecuted = false;
	RequestDelegate next = (HttpContext ctx) =>
	{
		wasExecuted = true;
		return Task.CompletedTask;
	};
	var middleware = new StatusMiddleware(next);
	await middleware.Invoke(context);
	
	Assert.True(wasExecuted);
}
```
---
Другой случай — когда в пути есть `"/ping"`. В этом случае должен быть сгенерирован соответствующий ответ с характеристиками:
- код ответа `200 OK`;
- в ответе должен быть заголовок `Content-Type:text/plain`;
- тело ответа должно содержать строку `"pong"`.

Так как каждая из характеристик — отдельное требование, обычно проверка каждой оформляется отдельным модульным тестом — так проще определить, какое требование не выполнено, если тест завершился неудачно. Однако для простоты рассмотрим все три проверки в одном тесте.
> [!important] Важно
> `DefaultHttpContext` использует `Stream.Null` для `Response.Body`, поэтому всё, что будет записано в `Body`, потеряется. Для нужд теста `Body` нужно заменить на `MemoryStream`.

```csharp
[Fact]
public async Task ReturnsPongBodyContent()
{
	var bodyStream = new MemoryStream();
	var context = new DefaultHttpContext();
	context.Response.Body = bodyStream;
	context.Request.Path = "/ping";
	RequestDelegate next = (ctx) => Task.CompletedTask;
	var middleware = new StatusMiddleware(next);
	
	await middleware.Invoke(context);
	
	string response;
	bodyStream.Seek(0, SeekOrigin.Begin);
	using(var stringReader = new StreamReader(bodyStream))
	{
		response = await stringReader.ReadToEndAsync();
	}
	Assert.Equal("pong", response);
	Assert.Equal("text/plain", context.Response.ContentType);
	Assert.Equal(200, context.Response.StatusCode);
}
```