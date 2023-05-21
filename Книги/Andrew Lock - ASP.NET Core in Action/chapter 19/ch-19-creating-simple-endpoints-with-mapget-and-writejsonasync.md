---
share: true
tags:
 - NET/ASPNETCore/endpoint
---
# Создание простых конечных точек с помощью MapGet и WriteJsonAsync
Помимо [[ch-19-creating-custom-endpoint-routing-component|преобразования]] компонента промежуточного ПО в конечную точку (с созданием выделенного конвейера) можно создавать конечные точки напрямую при помощи метода расширения `MapGet()`:
```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
	app.UseRouting();
	app.UseAuthentication();
	app.UseAuthorization();
	app.UseEndpoints(endpoints =>
	{
		endpoints.MapGet("/ping", (HttpContext ctx) => ctx.Response.WriteAsync("pong"));
		endpoints.MapRazorPages();
		endpoints.MapHealthChecks("/healthz");
	});
}
```

Метод `MapGet()` принимает [[route-template|шаблон маршрута]] первым параметром. Также существуют аналогичные методы расширения для других HTTP-методов, такие как `MapPost()` и `MapPut()`.

В ASP.NET Core 5.0 появились дополнительные методы расширения, позволяющие легко читать и писать JSON при помощи сериализатора System.Text.Json.
Напишем простую конечную точку `"/echo"`, считывающую JSON из тела запроса и пишущую его в ответ.
```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
	app.UseRouting();
	app.UseAuthentication();
	app.UseAuthorization();
	app.UseEndpoints(endpoints =>
	{
		endpoints.MapGet("/echo", (HttpContext ctx) =>
		{
			var model = await ctx.Request.ReadFromJsonAsync<MyCustomType>();
			await ctx.Response.WriteAsJsonAsync(model);
		});
		endpoints.MapRazorPages();
		endpoints.MapHealthChecks("/healthz");
	});
	
}
```
> [!tip] Совет
> Используйте методы расширения `Map*` и вспомогательные методы работы с JSON, если всё, что вам нужно — некое простое API. Если требуются дополнительные функции, такие как [[ch-6-validating-on-server-for-safety|валидация]], [[model-binding|привязка модели]] или интеграция с OpenAPI, используйте контроллеры веб-API. 

Также нужно учитывать, что маршрутизация конечных точек может работать существенно медленнее, чем простое [[ch-19-branching-middleware-pipelines-with-map-extension|ветвление]], поэтому в некоторых случаях лучше избегать маршрутизации конечных точек.