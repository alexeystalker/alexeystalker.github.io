---
share: true
tags:
 - NET/ASPNETCore/routing
 - NET/ASPNETCore/endpoint
---
# Маршрутизация в ASP.NET Core
Начиная с версии 3.0, в ASP.NET Core используется *маршрутизация конечных точек*.
## Использование маршрутизации конечных точек в ASP.NET Core
Маршрутизация [[endpoint|конечных точек]] реализуется с помощью двух компонентов:
- `EndpointMiddleware` для *регистрации* конечных точек в системе маршрутизации;
- `EndpointRoutingMiddleware` выбирает, какая из конечных точек, зарегистрированных `EndpointMiddleware`, должна выполняться для данного запроса. Чтобы легче их различать, будем называть его `RoutingMiddleware`.

Чтобы зарегистрировать конечные точки, вызовите метод `UseEndpoints()` в методе `Configure()` файла Startup.cs:
```csharp
public void Configure(IApplicationBuilder app)
{
	app.UseRouting(); //1
	app.UseEndpoints(endpoints => //2
	{
		endpoints.MapRazorPages(); //3
		endpoints.MapHealthChecks("/healthz"); //4
		endpoints.MapGet("/test", async context => //5
		{
			await context.Response.WriteAsync("Hello, World!");
		});
	});
}
```
Здесь:
1. добавляем компонент `EndpointRoutingMiddleware` в конвейер;
2. добавляем компонент `EndpointMiddleware` в конвейер и определяем конфигурацию;
3. регистрируем все страницы Razor;
4. регистрируем конечную точку проверки работоспособности на маршруте `/healthz`;
5. регистрируем простую конечную точку, возвращающую "Hello, World!" на маршруте `/test`

Каждая конечная точка связана с [[route-template|шаблоном маршрута]], который определяет, с какими URL-адресами конечная точка должна совпасть.

`EndpointMiddleware` хранит зарегистрированные маршруты в словаре, который используется `RoutingMiddleware` для поиска подходящих конечных точек. Если такая точка находится, указание на нее прикрепляется к объекту `HttpContext`. Когда запрос доходит до `EndpointMiddleware`, компонент выполняет выбранную точку.
Если `RoutingMiddleware` не находит подходящий запросу шаблон, конечная точка не выбирается, запрос передается дальше по конвейеру. `EndpointMiddleware` игнорирует такой запрос, и, если компонентов после `EndpointMiddleware` нет, выполняется фиктивный компонент и возвращается ответ 404 Not Found.
![[Pasted image 20220127182804.png]]
Видно преимущество разделения `RoutingMiddleware` и `EndpointMiddleware`: все компоненты, размещенные после `RoutingMiddleware`, могут видеть, какая конечная точка будет выполнена. Это полезно, например, при использовании компонента `AuthorizationMiddleware`, которому необходимо знать выбранную конечную точку для [[ch-15-authorization|проверки прав доступа]].
#### [[ch-5-convention-based-vs-attribute|Маршрутизация на основе соглашений и маршрутизация на основе атрибутов]]
#### [[ch-5-routing-to-razor-pages|Маршрутизация и страницы Razor]]
