---
share: true
tags:
 - NET/ASPNETCore/Identity
---
# Изучение шаблона в обозревателе решений
![[Pasted image 20220605204146.png]]
1. Пользовательский интерфейс по умолчанию содержится в **Areas/Identity** — там он создаётся пакетом Microsoft.AspNetCore.Identity.UI;
> [!Info]- Области
> *Области* используются для группировки страниц Razor Pages в отдельные иерархии для организационных целей. [Подробнее](https://docs.microsoft.com/en-us/aspnet/core/mvc/controllers/areas?view=aspnetcore-5.0)
2. Шаблон включает в себя `DbContext` от EF Core и миграции, чтобы сконфигурировать БД [[ch-14-asp-net-core-identity-data-model|подробнее]];
3. Папка **Pages** идентична шаблону без аутентификации, с дополнительным представлением **\_LoginPartial**. В нем можно увидеть, как маршрутизация работает с областями, объединяя путь к странице Razor с параметром маршрута `{area]` с помощью тег-хелперов:
	```html
	<a asp-area="Identity" asp-page="/Account/Login">Login</a>
	```

Также добавилась настройка в `ConfigureServices`:
```csharp
public void ConfigureServices(IServiceCollection services)
{
	var connectionString = Configuration.GetConnectionString("DefaultConnection");
	services.AddDbContext<ApplicationDbContext>(options =>
		options.UseSqlServer(connectionString));
	services.AddDefaultIdentity<IdentityUser>(options => options.SignIn.RequireConfirmedAccount = true)
		.AddEntityFrameworkStores<ApplicationDbContext>();
	
	services.AddRazorPages();
}
```
Здесь метод расширения `AddDefaultIdentity()` выполняет несколько функций:
- добавляет основные сервисы ASP.NET Core Identity;
- настраивает тип пользователя приложения как `IdentityUser`. При необходимости можно расширить этот тип;
- добавляет страницы Razor Pages пользовательского интерфейса по умолчанию для регистрации, входа и управления пользователями;
- настраивает поставщиков токенов для создания токенов подтверждения по электронной почте.

 В файле Startup добавляется строчка в методе `Configure`: `app.UseAuthentication()`. Мы добавляем компонент `AuthenticationMiddleware`, как показано на рисунке:
![[Pasted image 20220601195031.png]]
Очень важно расположение этого компонента — его нужно размещать после метода `UseRouting()` и перед методом `UseAuthorization()` и `UseEndpoints()`.
```csharp
public static void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
	app.UseStaticFiles();
	
	app.UseRouting();
	
	app.UseAuthentication();
	app.UseAuthorization();
	
	app.UseEndpoints(endponts => endpoints.MapRazorPages());
}
```