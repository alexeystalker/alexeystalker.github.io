---
share: true
tags:
 - NET/ASPNETCore/routing
 - NET/Razor
---
# Настройка соглашений с помощью Razor Pages
По умолчанию ASP.NET Core генерирует URL-адреса, которые точно соответствуют именам файлов страниц Razor. Однако, это поведение можно изменить, настроив объект `RouteOptions` в файле Startup.cs.
```csharp
public void ConfigureServices(IServiceCollection services)
{
	services.AddRazorPages();
	services.Configure<RouteOptions>(options =>
	{
		options.AppendTrailingSlash = true;
		options.LowercaseUrls = true;
		options.LowercaseQueryStrings = true;
	})
}
```
А вот как задать преобразование из PascalCase в kebab-case. Сначала нужно реализовать класс преобразователь:
```csharp
public class KebabCaseParameterTransformer : IOutboundParameterTransformer
{
	public string TransformOutbound(object value)
	{
		if(value == null) return null;
		return Regex.Replace(value.ToString(), "([a-z])([A-Z])", "$1-$2").ToLower();
	}
}
```
Далее регистрируем преобразователь:
```csharp
public void ConfigureServices(IServiceCollection services)
{
	services.AddRazorPages()
		.AddRazorPagesOptions(opts =>
		{
			opts.Conventions.Add(new PageRouteTransformerConvention(new KebabCaseParameterTransformer()));
		});
}
```
Также можно добавить *дополнительный* маршрут для страницы:
```csharp
public void ConfigureServices(IServiceCollection services)
{
	services.AddRazorPages()
		.AddRazorPagesOptions(opts =>
		{
			opts.Conventions.AddPageRoute("/Search/Products/StartSearch", "/search-products");
		});
}
```
Вместо замены маршрута (если бы мы [[ch-5-customizing-route-templates#Добавление сегмента в шаблон маршрута|редактировали]] [[razor-directive|директиву]] `@page`), мы добавляем дополнительный.
Все возможности по изменению соглашений доступны в [документации](https://docs.microsoft.com/en-us/aspnet/core/razor-pages/razor-pages-conventions?view=aspnetcore-5.0).
