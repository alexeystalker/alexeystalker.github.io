---
share: true
tags:
 - NET/MVC
 - API
---
# Создание первого проекта веб-API
Для того, чтобы создать проект веб-API, можно воспользоваться процедурой New Project в Visual Studio, аналогично описанному [[ch-2-creating-your-first-app|здесь]]. Нужно создать приложение ASP.NET Core, а в диалоговом окне нужно выбрать ASP.NET Core Web API.
Также можно использовать интерфейс командной строки:
```sh
dotnet new webapi -o WebApplication1
```
Конфигурация проекта происходит в файле Startup.cs, при этом в методе `ConfigureServices` вместо `AddRazorPages()` используется `AddControllers()`. Кроме того, в методе `UseEndpoints()` используется метод `MapControllers()`.
```csharp
public class Startup
{
	public void ConfigureServices(IServiceCollection services)
	{
		services.AddControllers();
		services.AddSwaggerGen(c =>
		{
			c.SwaggerDoc("v1", new OpenApiInfo 
			{
				Title = "DefaultApiTemplate",
				Version = "v1"
			});
		});
	}
	
	public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
	{
		if(env.IsDevelopment())
		{
			app.UseDevelopmentExceptionPage();
			app.UseSwagger();
			app.UseSwaggerUI(c => c.SwaggerEndpoint(
				"/swagger/v1/swagger.json", "DefaultApiTemplate v1"));
		}
		
		app.UseHttpsRedirection();
		app.UseRouting();
		app.UseAuthorization();
		
		app.UseEndpoints(endpoints => 
		{
			endpoints.MapControllers();
		});
	}
}
```
Также видно, что добавляются сервисы и конечные точки Swagger.
Вышеприведенный файл инструктирует приложение найти все контроллеры API и настроить их в `EndpointMiddleware`. Каждый [[action-method|метод действия]] становится конечной точкой.
Вот пример простого контроллера веб-API:
```csharp
[ApiController]
public class FruitController : ControllerBase
{
	List<string> _fruit = new List<string>
	{
		"Pear", "Lemon", "Peach"
	};
	
	[HttpGet("fruit")]
	public IEnumerable<string> Index()
	{
		return _fruit;
	}
}
```
Здесь: для применения общих соглашений добавляется атрибут `[ApiController]`, сам класс контроллера наследуется от `ControllerBase` для возможного использования вспомогательных методов для генерации результатов. Атрибут `[HttpGet("fruit")]` указывает, что шаблоном маршрута для этого метода является `"fruit"`, HTTP-метод — GET.
Метод `Index()` возвращает непосредственно данные, однако так делать не обязательно. Вместо этого можно вернуть объект `ActionResult`.
Добавим в наш контроллер еще один метод:
```csharp
[HttpGet("fruit/{id}")]
public ActionResult<string> View(int id)
{
	if(id >= 0 && id < _friut.Count)
	{
		return _fruit[id];
	}
	return NotFound();
}
```
Здесь, если фрукта с запрошенным индексом не существует, вернется ответ с кодом 404. Иначе — 200 с запрошенным результатом. Вместо прямого возвращения данных можно было явно использовать вспомогательный метод `Ok(_fruit[id])`.

Как только `ActionResult` возвращается из контроллера, он сериализуется в соответствующий ответ.