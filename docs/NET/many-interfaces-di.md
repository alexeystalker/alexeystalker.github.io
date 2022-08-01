---
share: true
tags: [NET/DI]
---
# Как зарегистрировать одну реализацию с несколькими интерфейсами
В стандартном ASP.NET Core DI  ситуация, когда нужно зарегистрировать одну реализацию нескольких разных интерфейсов, решается, на мой взгляд, не слишком тривиально.
Пусть есть класс `SomeService`, реализующий интерфейсы `ISomeService` и `ISomeOtherService`, то есть
```csharp
public class SomeService : ISomeService, IsomeOtherService 
{
...
}
```
И нам нужно зарегистрировать реализации этих интерфейсов отдельно. 
Вот что нам нужно сделать:
```csharp
public class Startup
{
	...
	public void ConfigureServices(IServiceCollection services)
	{
		...
		services.AddSingleton<SomeService>(); //Сначала регистрируем сам сервис
		services.AddSingleton<ISomeService>(ctx => 
			ctx.GetRequiredService<SomeService>()); //Затем регистрируем первый интерфейс с лямбдой вызова реализации сервиса
		services.AddSingleton<ISomeOtherService>(ctx =>
			ctx.GetRequiredService<SomeService>()); //А теперь второй интерфейс
		...
	}
}
	
```
В примере у нас синглтоны, но можно использовать и другие жизненные циклы.
## Ссылки
https://andrewlock.net/how-to-register-a-service-with-multiple-interfaces-for-in-asp-net-core-di/
