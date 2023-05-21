---
share: true
tags:
 - NET/worker-service
---
# Создание сервиса рабочей роли из шаблона
Для того, чтобы создать проект воркера, в VS выбираем **File > New > Project > Worker Service**. То же самое можно сделать, используя командную строку: `dotnet new worker`.
Теперь приложение надо настроить. Для примера настроим приложение, [[ch-22-running-background-tasks-on-a-timer|обновляющее курсы валют по таймеру]].
```csharp
public class Program
{
	public static void Main(string[] args)
	{
		CreateHostBuilder(args).Build().Run();
	}
	
	public static IHostBuilder CreateHostBuilder(string[] args) =>
		Host.CreateDefaultBuilder(args)
			.ConfigureServices((hostContext, services) =>
			{
				services.AddHttpClient<ExchangeRatesClient>();
				services.AddHostedService<ExchangeRatesHostedService>();
				services.AddDbContext<AppDbContext>(options =>
					options.UseSqlite(
						hostContext.Configuration
							.GetConnectionString("SqliteConnection")));
			});
}
```
> [!tip] Совет
> При помощи методов `IHostBuilder` можно контролировать, когда именно будут запускаться ваши фоновые задачи, автор написал об этом [пост в блоге](https://andrewlock.net/controlling-ihostedservice-execution-order-in-aspnetcore-3/)

Если мы посмотрим на .csproj файл проекта, мы увидим следующие сходные с проектом ASP.NET Core моменты:
- оба типа приложений должны указывать `<TargetFramework>`;
- оба типа приложений используют систему конфигурации, поэтому в воркерах можно использовать `<UserSecretsId>` также, как описано [[ch-11-storing-config-secrets-safely|здесь]];
- оба типа приложений должны явно добавлять ссылки на NuGet-пакеты EF Core.

Наряду с этим, есть и различия:
- атрибут `Sdk` элемента `Project` для воркера выглядит так: `Microsoft.NET.Sdk.Worker`, тогда как для ASP.NET Core — `Microsoft.NET.Sdk.Web`;
- сервис рабочей роли *должен* явно ссылаться на пакет Microsoft.Extensions.Hosting;
- возможно, потребуется явно добавить зависимости, неявно включённые в ASP.NET Core. Например - Microsoft.Extensions.Http, предоставляющий `IHttpClientFactory`[^1].

[^1]: О `IHttpClientFactory` написано [[ch-21-using-ihttpclientfactory-to-manage-httpclienthandler-lifetime|здесь]].