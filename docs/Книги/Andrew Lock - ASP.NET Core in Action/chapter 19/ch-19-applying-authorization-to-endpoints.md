---
share: true
tags:
 - NET/ASPNETCore/endpoint
 - NET/authorization
---
# Применение авторизации к конечным точкам
Одним из основных преимуществ маршрутизации конечных точек является возможность легко применять [[ch-15-authorization|авторизацию]] к конечной точке. Для контроллеров веб-API и страниц Razor для этого используется атрибут `[Authorize]`.
Для других конечных точек, таких как [[ch-19-creating-custom-endpoint-routing-component|ping-pong]], можно использовать метод `RequireAuthorization()` при добавлении точки в приложение:
```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
	app.UseRouting();
	app.UseAuthentication();
	app.UseAuthorization();
	
	app.UseEndpoints(endpoints =>
	{
		endpoints.MapPingPong("/ping")
			.RequireAuthorization();
		endpoints.MapRazorPages();
		endpoints.MapHealthChecks("/healthz")
			.RequireAuthorization("HealthCheckPolicy");
	});
}
```
Здесь показано два примера применения авторизации к конечным точкам:
- `RequireAuthorization()` — равносильно применению атрибута `[Authorize]`;
- `RequireAuthorization(policy)` — если указано имя политики, будет использоваться выбранная политика авторизации. Политика должна быть [[ch-15-creating-a-policy-with-custom-requirement-and-handler|настроена]] в `ConfigureServices`. Эквивалентно применению атрибута `[Authorize("HealthCheckPolicy")]`.

Если авторизация применяется [[ch-15-preventing-anonymous-users-from-accessing-your-application|глобально]], можно создать брешь в глобальной политике при помощи метода `AllowAnonymous()`, например
```csharp
endpoints.MapPingPong("/ping").AllowAnonymous();
```
Что эквивалентно использованию атрибута `[AllowAnonymous]`.