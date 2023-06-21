---
share: true
tags:
 - NET/ASPNETCore/filters
 - NET/MVC
 - NET/Razor
---
# Добавляем фильтры к действиям, контроллерам, страницам Razor Pages и глобально
Добавлять фильтры можно тремя способами:
- *к конкретному действию или странице Razor* — фильтр будет выполняться только при обращении к определенному действию или странице;
- *к контроллеру* — фильтр будет применен к всем действиям контроллера;
- *глобально* — фильтры применяются ко всем действиям или страницам.

Уровень, на котором применяется фильтр, называется его [[filter-scope|областью дейстия]]
Рассмотрим, как применять [[ch-13-creating-a-simple-filter|фильтр]] к разным областям действия.

Применим фильтры к одному действию:
```csharp
public class RecipeController ; ControllerBase
{
	[LogResourceFilter]
	public IActionResult Index()
	{
		return Ok();
	}
	
	public IActionResult View()
	{
		return Ok();
	}
}
```
Здесь фильтр будет выполняться только при вызове метода действия `Index()`.

Теперь, чтобы применить фильтр ко всем действиям контроллера, применим фильтр непосредственно к контроллеру:
```csharp
[LogResourceFilter]
public class RecipeController : ControllerBase
{
	public IActionResult Index()
	{
		return Ok();
	}
	
	public IActionResult View()
	{
		return Ok();
	}
}
```
Здесь фильтр будет применен ко всем действиям контроллера.

Теперь применим фильтр к странице Razor. Невозможно применить фильтр только к одному обработчику на странице — применение возможно только к странице целиком:
```csharp
[LogResourceFilter]
public class IndexModel : PageModel
{
	public void OnGet() {}
	public void OnPost() {}
}
```

Предположим теперь, что есть фильтр, который необходимо применить ко всем контроллерам и/или страницам Razor. Для этого нужно добавить фильтры при конфигурировании сервисов MVC (показано три аналогичных друг другу способа):
```csharp
public class Startup
{
	public void ConfigureServices(IServiceCollection services)
	{
		services.AddControllers(options =>
		{
			options.Filters.Add(new LogResourceFilter());
			options.Filters.Add(typeof(LogResourceFilter));
			options.Filters.Add<LogResourceFilter>();
		})
	}
}
```

При использовании Razor Pages код немного отличается, необходимо использовать метод расширения `AddMvcOptions()`:
```csharp
public class Startup
{
	public void ConfigureServices(IServiceCollection services)
	{
		services.AddRazorPages().AddMvcOptions(options =>
		{
			options.Filters.Add(new LogResourceFilter());
			options.Filters.Add(typeof(LogResourceFilter));
			options.Filters.Add<LogResourceFilter>();
		})
	}
}
```
