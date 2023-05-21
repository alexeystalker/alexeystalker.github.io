---
share: true
tags:
 - NET/ASPNETCore
 - NET/configuration
---
# Конфигурирование приложения с помощью метода CreateDefaultBuilder
Рассмотрим, где происходит загрузка конфигурации от [[configuration-provider|поставщиков]]. Для приложений ASP.NET Core 5.0, созданных с использованием шаблонов по умолчанию, это всегда происходит внутри метода `Host.CreateDefaultBuilder()` в файле Program.cs.
Как мы [[ch-2-program-cs|видели]], в ASP.NET Core 5.0 по умолчанию используется метод `CreateDefaultBuilder()`:
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
		});
}
```
Вот как выглядит этот метод:
```csharp
public static IHostBuilder CreateDefaultBuilder(string[] args)
{
	var builder = new HostBuilder()
		.UseContentRoot(Directory.GetCurrentDirectory())
		.ConfigureHostConfiguration(config => {/*настройка поставщика конфигурации*/})
		.ConfigureAppConfiguration((hostingContext, config) => {/*настройка поставщика конфигурации*/})
		.ConfigureLogging((hostingContext, logging) =>
		{
			logging.AddConfiguration(
				hostingCOntext.Configuration.GetSection("Logging"));
			logging.AddConsole();
			logging.AddDebug();
		})
		.UseDefaultServiceProvider((context, options) =>
		{
			var isDevelopment = context.HostingEnvironment.IsDevelopment();
			options.ValidateScopes = isDevelopment;
			options.ValidateOnBuild = isDevelopment;
		});
	return builder;
}
```
Здесь:
- `UseContentRoot`: метод сообщает приложению, в каком каталоге можно найти любую конфигурацию или посмотреть файлы, которые потребуются позже;
- `ConfigureHostConfiguration`: место, где приложение определяет, в каком окружении (`HostingEnvironment`) оно работает[^1];
- `ConfigureAppConfiguration`: конфигурирует настройки приложения[^2];
- `ConfigureLogging`: конфигурирует параметры логирования[^3];
- `UseDefaultServiceProvider`: настраивает приложение на использование встроенного [[di-container|контейнера внедрения зависимостей]].

[^1]:[[ch-11-identifying-hosting-environment|Подробнее]]
[^2]:[[ch-11-adding-configuration-provider-in-program-cs|Подробнее]]
[^3]:[[ch-17-changing-log-verbosity-with-filtering|Подробнее]] 