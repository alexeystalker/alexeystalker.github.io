---
share: true
tags:
 - NET/configuration
---
# Добавление поставщика конфигурации в файле Program.cs
При использовании метода `CreateDefaultBuilder()` ([[ch-11-configuring-with-createdefaultbuilder|см.]]) по умолчанию устанавливаются следующие поставщики конфигурации:
- *поставщик файлов JSON* — загружает настройки из файла в формате JSON, appsettings.json, а также загружает дополнительные настройки из файла **appsettings.ENVIRONMENT.json**;[^1]
- *пользовательские секреты* — загружает секреты;
- *переменные окружения* — загружает переменные окружения в качестве переменных конфигурации; подходит для хранения секретов в промышленном окружении;
- *аргументы командной строки* — использует значения, переданные при запуске приложения.

Можно изменить этот набор при помощи дополнительного вызова `ConfigureAppConfiguration()`:
```csharp
public class Program
{
	public static void Main(string[] args)
	{
		CreateHostBuilder(args).Build().Run();
	}
	public static IHostBuilder CreateHostBuilder(string[] args) =>
		Host.CreateDefaultBuilder(args)
		.ConfigureAppConfiguration(AddAppConfiguration)
		.ConfigureWebHostDefaults(webBuilder =>
	    {
			webBuilder.UseStartup<Startup>();
	    });
	//Также можно было оформить в виде лямбды
	public static void AddAppConfiguration(HostBuilderContext hostingContext, IConfigurationBuilder config)
	{
		config.Sources.Clear();
		config.AddJsonFile("appsettings.json", optional: true);
	}
}
```
Здесь мы очистили набор источников, затем добавили источник — файл **appsettings.json**. Параметр `optional` отвечает за пропуск ненайденных файлов (в противном случае будет исключение `FileNotFoundException`).
Далее, при вызове метода `Build()` у `IHostBuilder` будет сгенерирован объект `IConfiguration`, который затем регистрируется в [[di-container|контейнере зависимостей]]. Обычно он внедряется в конструктор `Startup`, чтобы можно было продолжить конфигурирование в зависимости от параметров конфигурации:
```csharp
public class Startup
{
	public Startup(IConfiguration config)
	{
		Configuration = config;
	}
	public IConfiguration Configuration { get; }
}
```

`Iconfiguration` хранит конфигурацию в виде набора пар “ключ-значение”.
Чтобы прочитать значение, используется стандартный синтаксис словаря, например:
```csharp
var zoomlevel = Configuration["MapSettings:DefaultZoomLevel"];
```
`:` используется, чтобы выделить отдельный раздел. Можно спуститься на несколько уровней:
```csharp
Configuration["MapSettings:DefaultLocation:Latitude"];
```
Если ключ отсутствует, будет возвращено значение `null`;
Чтобы получить целый раздел конфигурации как единый объект, используем метод `GetSection()`:
```csharp
Configuration.GetSection("MapSettings")["DefaultLocation:Latitude"];
```

[^1]: [[ch-11-loading-environment-specific-config-files|Подробнее]]