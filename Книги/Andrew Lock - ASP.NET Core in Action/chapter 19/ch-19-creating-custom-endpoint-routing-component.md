---
share: true
tags:
 - NET/ASPNETCore/endpoint
---
# Создание специального компонента маршрутизации конечных точек
Как уже [[ch-5-routing-in-asp-net-core|было сказано]], маршрутизация [[endpoint|конечных точек]] разделяет процесс на два этапа, реализуемых отдельными компонентами:
- `RoutingMiddleware` — использует входящий запрос для выбора конечной точки. Предоставляет метаданные о выбранной конечной точке в `HttpContext` (например, требования к авторизации, [[ch-15-authorization-in-asp-net-core|задаваемые]] при помощи `[Authorize]`);
- `EndpointMiddleware` — выполняет выбранную конечную точку для генерации ответа.

Преимущество двухэтапного процесса в том, что можно разместить промежуточное ПО между этапом выбора конечной точки и этапом выполнения.
Пусть мы хотим применить авторизацию к простой конечной точке ping-pong, построенной [[ch-19-branching-middleware-pipelines-with-map-extension|здесь]]. Для этого воспользуемся маршрутизацией конечных точек.
Сперва создадим класс `PingPongMiddleware` (согласно подходу [[ch-19-building-custom-middleware-component|отсюда]]):
```csharp
public class PingPongMiddleware
{
	public PingPongMiddleware(RequestDelegate next) {}
	
	public async Task Invoke(HttpContext context)
	{
		context.Response.ContentType = "text/plain";
		await context.Response.WriteAsync("pong");
	}
}
```
Теперь преобразуем компонент в конечную точку. Для этого *создадим мини-конвейер* внутри лямбда-функции `UseEndpoints()`, используя метод `CreateApplicationBuilder()`:
```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
	app.UseRouting();
	app.UseAuthentication();
	app.UseAuthorization();
	
	app.UseEndpoints(endpoints =>
	{
		var endpoint = endpoints
			.CreateApplicationBuilder()
			.UseMiddleware<PingPongMiddleware>()
			.Build();
		
		endpoints.Map("/ping", endpoint);
		endpoints.MapRazorPages();
		endpoints.MapHealthChecks("/healthz");
	});
}
```
> [!tip] Совет
> Заметьте, что `Map()` в `IEndpointRouteBuilder` создаёт новую конечную точку, ассоциированную с маршрутом. Эта функция концептуально отличается от [[ch-19-branching-middleware-pipelines-with-map-extension|описанной ранее]], которая используется для ветвления конвейера.

Также целесообразно вынести создание конечной точки в отдельный метод расширения:
```csharp
public static class EndpointRouteBuilderExtensions
{
	public static IEndpointConventionBuilder MapPingPong(this IEndpointRouteBuilder endpoints, string route)
	{
		var pipeline = endpoints
			.CreateApplicationBuilder()
			.UseMiddleware<PingPongMiddleware>()
			.Build();
		
		return endpoints
			.Map(route, pipeline)
			.WithDisplayName("Ping-pong");
	}
}
```
Теперь метод `UseEndpoints()` станет проще:
```csharp
app.UseEndpoints(endpoints => 
{
	endpoints.MapPingPong("/ping");
	endpoints.MapRazorPages();
	endpoints.MapHealthChecks("/healthz");
});
```
> [!tip] Совет
> Здесь мы использовали простой маршрут `"/ping"`, но можно использовать [[route-template|шаблоны маршрута]] с параметрами, например `"/ping/{count}"`. Подробности см. в [блоге автора](https://andrewlock.net/accessing-route-values-in-endpoint-middleware-in-aspnetcore-3/)
