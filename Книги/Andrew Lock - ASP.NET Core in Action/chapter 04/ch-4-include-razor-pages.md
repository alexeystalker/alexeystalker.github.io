---
share: true
tags:
 - NET/Razor
 - NET/dotnet
---
# Добавление Razor Pages в приложение
Как добавить Razor Pages в приложение.
1. Создадим проект веб-приложения ASP.NET Core в VS или из командной строки `dotnet new web`
2. Добавляем сервисы Razor Pages в метод `ConfigureServices()` файла Startup.cs
	```csharp
	public void ConfigureServices(IServiceCollection services)
	{
		services.AddRazorPages();
	}
	```
3. Заменяем базовую конечную точку в конце конвейера на `MapRazorPages()`
	```csharp
	public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
	{
		app.UseRouting();
		app.UseEndpoints(endpoints =>
						 {
							 endpoints.MapRazorPages();
						 });
	}
	```
4. В корень проекта добавляем новую папку Pages.
5. ПКМ на папку Pages, выбираем **Add > RazorPage**
6. Выбираем **Razor Page — Empty** и называем страницу Index.cshtml
7. Откроем файл Index.cshtml и заменим содержимое на это:
	```razor
	@page
	@model AddingRazorPagesToEmptyProject.IndexModel
	@{
		Layout = null;
	}
	<!DOCTYPE html>
	<html>
		<head>
			<meta name="viewport" content="width=device-width" />
			<title>Index</title>
		</head>
		<body>
			<h1>Hello World!</h1>
		</body>
	</html>
	```
	
Шаги 4–6 можно также выполнить, выполнив команду `dotnet new page -n Index -o Pages/` в каталоге проекта.