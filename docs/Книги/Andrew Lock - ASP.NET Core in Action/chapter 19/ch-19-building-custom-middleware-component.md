---
share: true
tags:
 - NET/ASPNETCore/middleware
---
# Создание специального компонента промежуточного ПО
При создании промежуточного ПО [[ch-19-adding-to-pipeline-with-use-extension|при помощи метода]] `Use()` возникают некоторые ограничения. Например, нельзя использовать внедрение зависимостей с жизненным циклом Scoped, а также тестирование этого кода может быть затруднено. Поэтому вместо того, чтобы использовать метод `Use()`, промежуточное ПО инкапсулируется в класс.
ASP.NET Core использует рефлексию (reflection) для вызова метода промежуточного ПО. Поэтому нет необходимости наследовать классы промежуточного ПО от особого базового класса или реализовывать некоторый интерфейс. Однако, нужно выполнить некоторые требования, а именно — у класса промежуточного ПО должен быть конструктор, принимающий объект `RequestDelegate` — остальную часть конвейера, и функция `Invoke` с сигнатурой
```csharp
public Task Invoke(HttpContext context);
```
Преобразуем метод подстановки заголовков в отдельный класс.
```csharp
public class HeadersMiddleware
{
	private readonly RequestDelegate _next;
	public HeadersMiddleware(RequestDelegate next)
	{
		_next = next;
	}
	public async Task Invoke(HttpContext context)
	{
		context.Response.OnStarting(() =>
		{
			context.Response.Headers["X-Content-Type-Options"] = "nosniff";
			return Task.CompletedTask;
		});
		
		await _next(context);
	}
}
```
После этого можно добавить компонент в приложение строкой
```csharp
app.UseMiddleware<HeadersMiddleware>();
```
Существует распространённая практика создавать вспомогательные методы расширения для использования классов промежуточного ПО (чтобы IntelliSense показывал этот метод в экземпляре `IApplicationBuilder`):
```csharp
public static class MiddlewareExtensions
{
	public static IApplicationBuilder UseSecurityHeaders(this IapplicationBuilder app)
	{
		return app.UseMiddleware<HeadersMiddleware>();
	}
}
```

В некоторых случаях при работе компонента может потребоваться использовать внедрение зависимостей для внедрения дополнительных сервисов. В вышеописанный класс можно внедрить сервисы с жизненным циклом Singleton в конструкторе, или сервисы с любым жизненным циклом в методе Invoke:
```csharp
public class ExampleMiddleware
{
	private readonly RequestDelegate _next;
	private readonly ServiceA _a;
	public ExampleMiddleware(RequestDelegate next, ServiceA a)
	{
		_next = next;
		_a = a;
	}
	public async Task Invoke(HttpContext context, ServiceB b, ServiceC c)
	{
		//Используем сервисы a, b и c
		//Если нужно, вызываем _next(context)
	}
}
```
