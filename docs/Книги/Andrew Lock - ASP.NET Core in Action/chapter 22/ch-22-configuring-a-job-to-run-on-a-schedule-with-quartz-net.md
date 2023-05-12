---
share: true
tags:
 - NET/worker-service
---
# Настройка запуска задания по расписанию с помощью Quartz.NET
[[ch-22-running-background-tasks-with-ihostedservice|Ранее]] мы создали экземпляр сервиса, который скачивает курсы обмена валют и сохраняет результаты в БД с помощью EF Core. Теперь реализуем аналогичную логику для запуска по расписанию.
Рассмотрим такую реализацию интерфейса `IJob`:
```csharp
public class UpdateExchangeRatesJob : IJob
{
	private readonly ILogger<UpdateExchangeRatesJob> _logger;
	private readonly ExchangeRatesClient _typedClient;
	private readonly AppDbContext _dbContext;
	public UpdateExchangeratesJob(
		ILogger<UpdateExchangeRatesJob> logger,
		ExchangeRatesClient typedClient,
		AppDbContext dbContext)
	{
		_logger = logger;
		_typedClient = typedClient;
		_dbContext = dbContext;
	}
	
	public async Task Execute(IJobExecutionContext context)
	{
		_logger.LogInformation("Fetching latest rates");
		var latestRates = await _typedClient.GetLatestRatesAsync();
		
		_dbContext.Add(latestRates);
		await _dbContext.SaveChangesAsync();
		
		_logger.LogInformation("Latest rates updated");
	}
}
```
Отметим некоторые моменты, отличные от кода [[ch-22-using-scoped-services-in-background-tasks|отсюда]]:
- `IJob` определяет только задачу, которую нужно выполнить; он не определяет информацию о времени;
- новый экземпляр `IJob` создаётся каждый раз при выполнении задания;
- зависимости с ЖЦ Scoped внедряются непосредственно в конструкторе.

Теперь зарегистрируем нашу задачу для выполнения, настроив триггер:
```csharp
public void ConfigureServices(IServiceCollection services)
{
	services.AddQuartz(q => 
	{
		q.UseMicrosoftDependencyInjectionScopedFactory();
		//создаём уникальный ключ задания
		var jobKey = new JobKey("Update exchange rates");
		//Добавляем наше задание в [[di-container|контейнер зависимостей]] и связываем его с ключом задания
		q.AddJob<UpdateExchangeRatesJob>(opts => opts.WithIdentity(jobKey));
		//добавляем и настраиваем триггер
		q.AddTrigger(opts => opts
			.ForJob(jobKey)
			.WithIdentity(jobKey.Name + " trigger")
			.StartNow()
			.WithSimpleSchedule(x => x
				.WithInterval(TimeSpen.FromMinutes(5))
				.RepeatForever()));
	});
	
	services.AddQuartzHostedService(q => q.WaitForJobsToComplete = true);
}
```
> [!tip] Совет
> Чтобы запретить создание более одного экземпляра задания одновременно (например, когда ещё не отработал предыдущий запуск), нужно декорировать реализацию `IJob` атрибутом `[DisallowConcurrentExecution]`.

Подробное описание параметров конфигурации триггера и примеры можно найти в [документации Quartz.NET](https://www.quartz-scheduler.net/documentation/)