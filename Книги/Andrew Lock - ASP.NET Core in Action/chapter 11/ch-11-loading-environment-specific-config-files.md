---
share: true
tags:
 - NET/configuration
 - NET/environment
---
# Загрузка файлов конфигурации для конкретного окружения
Значение `EnvironmentName` определяется до того, как будет создан `ConfigurationBuidler`, переданный в `ConfigureAppConfiguration`. Это означает, что можно менять поставщики, добавляемые в построитель, в зависимости от `EnvironmentName`.
Распространенный шаблон — наличие необязательного файла appsettings.ENVIRONMENT.json для конкретного окружения. Например:
```csharp
public class Program
{
	public static void AddAppConfiguration(
		HostBuilderContext hostingContext,
		IConfigurationBuilder config)
	{
		var env = hostingContext.HostingEnvironment;
		config
			.AddJsonFile(
				"appsettings.json",
				optional: false)
			.AddJsonFile(
				$"appsettings.{env.EnvironmentName}.json",
				optional: true);
	}
}
```
Таким образом, значения, специфичные для какого-либо окружения, будут перезаписывать значения из глобального файла appsettings.json.
Еще один распространенный шаблон - удаление или добавление поставщиков конфигурации в зависимости от приложения:
```csharp
public class Program
{
	public static void AddAppConfiguration(
		HostBuilderContext hostingContext,
		IConfigurationBuilder config)
	{
		var env = hostingContext.HostingEnvironment;
		config
			.AddJsonFile(
				"appsettings.json",
				optional: false)
			.AddJsonFile(
				$"appsettings.{env.EnvironmentName}.json",
				optional: true);
		if(env.IsDevelopment())
		{
			config.AddUserSecrets<Startup>();
		}
	}
}
```

Также в зависимости от окружения можно настроить конвейер промежуточного ПО. Так было [[ch-3-handling-errors-with-middleware#Просмотр исключений в окружении разработки DeveloperExceptionPage|организовано]] использование компонента `DeveloperExceptionPageMiddleware`:
```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
	if(env.IsDevelopment())
	{
		app.UseDeveloperExceptionPage();
	}
	else
	{
		app.UseExceptionHandler("/Error");
	}
	/*дальнейшая настройка*/
}
```