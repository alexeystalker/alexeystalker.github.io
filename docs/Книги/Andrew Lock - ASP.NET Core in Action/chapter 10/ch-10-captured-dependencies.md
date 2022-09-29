---
share: true
tags:
 - NET/ASPNETCore
 - NET/DI
---
# Следите за захваченными зависимостями
Предположим, мы настраиваем ЖЦ для `DataContext` и `Repository`:
- `DataContext` - scoped, так как он должен использоваться для одного запроса
- `Repository` - singleton, так как он потокобезопасный и не имеет собственного состояния.

Если мы поступим таким образом, мы получим *захваченную зависимость*. `DataContext`, использованный в `Repository` всегда будет одним и тем же, так как он будет сохраняться всё время жизни `Repository`.
Поэтому сервис должен использовать только те зависимости, жизненный цикл которых превышает или эквивалентен ЖЦ сервиса. Singleton сервис может использовать только singleton зависимости. Scoped сервис - только scoped и singleton. Transient - все три вида.

По умолчанию ASP.NET Core проверяет захватываемые зависимости на соответствие этому правилу только в окружении разработки, так как она влияет на производительность. Чтобы включить или отключить проверку независимо от окружения, нужно задать параметр `ValidateScopes` при создании `HostBuilder`:
```csharp
public class Program
{
	public static void Main(string[] args)
	{
		CreateHostBuilder(args).Build().Run();
	}
	
	public static IHostBuilder CreateHostBuilder(string[] args) =>
		Host.CreateDefaultBuilder(args)
		.ConfigureWebHostDefaults(webBuilder =>
		{
			webBuilder.UseStartup<Startup>();
		})
		.UseDeafultServiceProvider(options =>
		{
			options.ValidateScopes = true;
			options.ValidateOnBuild = true;
		})
}
```
В примере также включен параметр `ValidateOnBuild`, который отвечает за проверку всего [[dependency-graph|графа зависимостей]] на наличие регистрации всех требуемых зависимостей.