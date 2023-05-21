---
share: true
tags:
 - NET/logging
---
# Добавление нового поставщика журналирования в приложение
Чтобы добавить нового поставщика журналирования, выполним следующие действия.
1. Добавим Nuget-пакет поставщика, например NetEscapades.Extensions.Logging.RollingFile[^1]. Добавим либо при помощи Диспетчера пакетов в VS, либо командой
	```bash
	dotnet add package NetEscapades.Extensions.Logging.RollingFile
	```
1. Добавим поставщика методом `IHostBuilder.ConfigureLogging()`. Воспользуемся методом расширения `AddFile` из пакета поставщика:
	```csharp
	public class Program
	{
		public static void Main(string[] args)
		{
			CreateHostBuilder(args).Build.Run();
		}

		public static IHostBuilder CreateHostBuilder(string[] args) =>
			Host.CreateDefaultBuilder(args)
				.ConfigureLogging(builder => builder.AddFile())
				.ConfigureWebHostDefaults(webBuilder =>
				{
				  webBuilder.UseStartup<Startup>();
				});
	}
	```
	
	[^1]: Это пакет, написаный автором книги. Код доступен [здесь](https://github.com/andrewlock/NetEscapades.Extensions.Logging)
	
> [!note] Примечание
> Добавление нового поставщика не заменяет существующих. Так как мы использовали метод `CreateDefaultBuilder()`, поставщики консоли и отладки уже добавлены. Чтобы удалить их, нужно вызвать метод `builder.ClearProviders()` в начале лямбды в `ConfigureLogging()`. Или использовать специальный `HostBuilder`.

Стоит заметить, что использовать запись в файл в промышленном окружении может быть не лучшим вариантом (хотя это и лучше, чем использовать несуществующее окно консоли). Вместо этого лучше воспользоваться специальным агрегатором журналов. Несколько вариантов: Loggr (кажется, помер, сайт недоступен), [elmah.io](https://elmah.io) или [[datalust-seq|Seq]].