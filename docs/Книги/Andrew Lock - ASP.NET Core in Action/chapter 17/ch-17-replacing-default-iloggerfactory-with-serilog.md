---
share: true
tags:
 - NET/logging
---
# Замена ILoggerFactory по умолчанию на Serilog
Попробуем заменить дефолтную реализацию интерфейса `ILoggerFactory` на реализацию, использующую [[library-serilog|Serilog]].
Для того, чтобы использовать Serilog, заменим ILoggerFactory на специальную реализацию, содержащую одного поставщика, `SerilogLoggerProvider`. И уже этот поставщик будет записывать во все *получатели данных* Serilog.
Добавим одного получателя данных для записи в консоль. Для этого:
1. добавим все необходимые NuGet-пакеты;
2. создаем и конфигурируем логгер Serilog;
3. заменяем реализацию `ILoggerFactory` на `SerilogLoggerFactory`, используя метод `UseSerilog()`.

Первый шаг: добавим пакеты:
```bash
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Sinks.Console
```
Далее конфигурируем Serilog и заменяем `ILoggerFactory`:
```csharp
public class Program
{
	public static void Main(string[] args)
	{
		Log.Logger = new LoggerConfiguration() //создаём объект конфигурации
			.WriteTo.Console()
			.CreateLogger();
		try
		{
			CreateHostBuilder(args).Build().Run();
		}
		catch (Exception ex)
		{
			Log.Fatal(ex, "Host terminated unexpectedly");
		}
		finally
		{
			Log.CloseAndFlush();
		}
	}
	
	public static IHostBuilder CreateHostBuilder(string[] args) =>
		Host.CreateDefaultBuilder(args)
			.UseSerilog()
			.ConfigureWebHostDefaults(webBuilder =>
			{
				webBuilder.UseStartup<Startup>();
			})
}
```
Теперь можно записывать сообщения в журналы через методы статического класса `Log`.
> [!tip] Совет
> У Serilog есть много дополнительных функций. Например, возможность добавлять *обогатители* — для добавления дополнительной информации во все сообщения журналов. Подробно рекомендуемый способ настройки Serilog в ASP.NET Core разбирается в [посте](https://nblumhardt.com/2019/10/serilog-in-aspnetcore-3/) автора Serilog Николаса Блюмхардта.