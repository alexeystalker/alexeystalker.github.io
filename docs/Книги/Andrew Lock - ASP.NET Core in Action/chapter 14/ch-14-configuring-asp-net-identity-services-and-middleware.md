---
share: true
tags:
 - NET/ASPNETCore/Identity
---
# Настройка сервисов ASP.NET Core Identity и промежуточного ПО
Чтобы добавить ASP.NET Core Identity в существующее приложение, нужно добавить два NuGet-пакета:
- *Microsoft.AspNetCore.Identity.EntityFrameworkCore* — предоставляет все основные сервисы Identity и интеграцию с EF Core;
- *Microsoft.AspNetCore.Identity.UI* — предоставляет страницы Razor Pages для UI по умолчанию.

Далее добавляем конфигурацию в Startup.cs:
```csharp
public void ConfigureServices(IServiceCollection services)
{
	services.AddDbContext<AppDbContext>(options =>
		options.UseSqlServer(Configuration
								.GetConnectionString("DefaultConnection")));
	
	services.AddDefaultIdentity<ApplicationUser>(options => options
			.SignIn.RequireConfirmedAccount = true)
		.AddEntityFrameworkStores<AppDbContext>(); //Важно использовать существующий DbContext
	...
}
```
Здесь мы добавили тип `ApplicationUser` — наследника `IdentityUser`. Подробнее [[ch-14-updating-ef-core-data-model-to-support-identity|в следующем разделе]].
Также нужно настроить `AuthenticationMiddleware`:
```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
	app.UseStaticFiles(); //StaticFilesMiddleware не будет воспринимать запросы как аутентифицированные
	app.UseRouting();
	
	app.UseAuthentication();
	app.UseAuthorization();
	
	app.UseEndpoints(endpoints => endpoints.MapRazorPages());
}
```
