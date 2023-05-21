---
share: true
tags:
 - NET/logging
---
# Изменение избыточности сообщений журналов с помощью фильтрации
ASP.NET Core включает возможность фильтрации сообщений журнала *до* того, как они будут написаны, основываясь на сочетании трёх вещей:
- уровень сообщения журнала;
- категория средства ведения журнала;
- поставщик журналирования.

Можно создать правила, используя эти свойства, и для каждого конкретного журнала будет применяться наиболее конкретное правило, чтобы определить, следует ли записывать сообщение журнала. Например:
- *минимальный уровень сообщения по умолчнию — `Information`* — если другие правила не применяются, записываться будут только сообщения с уровнем `Information` или выше;
- *для категорий, начинающихся со слова Microsoft, минимальный уровень — `Warning`*;
- *для поставщика консоли минимальный уровень — `Error`*.

Обычно набор правил журналирования определяется в многоуровневой конфигурации, описанной [[ch-11-configuring-application|в главе 11]]. Для этого нужно вызвать метод `AddConfiguration` при настройке журналирования (или использовать метод `CreateDefaultBuilder()`). Вот как добавить правила конфигурации при настройке собственного `HostBuilder`:
```csharp
public class Program
{
	public static void Main(string[] args)
	{
		CreateHostBuilder(args).Build().Run();
	}
	
	public static IHostBuilder CreateHostBuilder(string[] args) =>
		new HostBuilder()
			.UseContentRoot(Directory.GetCurrentDirectory())
			.ConfigureAppConfiguration(config => config.AddJsonFile("appsettings.json"))
			.ConfigureLogging((ctx, builder) =>
			{
				builder.AddConfiguration(ctx.Configuration.GetSection("Logging"));
				builder.AddConsole();
			})
			.ConfigureWebHostDefaults(webBuilder =>
			{
				webBuilder.UseStartup<Startup>();
			})
}
```
Здесь мы загрузили всю конфигурацию из файла appsettings.json. Конфигурация журналирования взята из секции `"Logging"` объекта `IConfiguration`, доступного при вызове метода `ConfigureLogging()`.

> [!tip] Совет
> Как было [[ch-11-loading-environment-specific-config-files|показано ранее]], можно загружать параметры конфигурации из разных источников в зависимости от `IHostingEnvironment`. Распространённой практикой является включение параметров журналирования для промышленного окружения в appsettings.json, а переопределений для локального окружения разработки — в файл appsettings.Development.json.

Определим описанные выше правила в конфигурационном файле:
```json
{
	"Logging": {
		"LogLevel": {
			"Default": "Information",
			"Microsoft": "Warning"
		},
		"Console": {
			"LogLevel": {
				"Default": "Error"
			}
		}
	}
}
```

> [!warning] Внимание
> Правила фильтрации сообщений журналов не объединяются; выбирается одно правило. Включение правил для конкретного поставщика переопределит глобальные правила для конкретных категорий.

