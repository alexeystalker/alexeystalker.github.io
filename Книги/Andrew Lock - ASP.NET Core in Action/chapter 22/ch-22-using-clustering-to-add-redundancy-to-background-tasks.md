---
share: true
tags:
 - NET/worker-service
---
# Использование кластеризации для добавления избыточности в фоновые задачи
Предположим, у нас есть необходимость масштабировать наше приложение, выполняющее фоновые задачи — запустить, скажем, десять его экземпляров. Но мы не можем просто запустить несколько экземпляров приложения — все они будут независимо выполнять свои задания по одному расписанию, например, раз в пять минут, начиная со старта приложения. По многим причинам это может оказаться недопостимым.
Одним из вариантов решения этой проблемы будет использование *кластеризации*. Так, Quartz.NET обеспечивает кластеризацию, используя БД в качестве хранилища описания заданий и триггеров, включая время последнего срабатывания триггера.
Когда триггер указывает, что задание необходимо выполнить, планировщики каждой реплики приложения пытаются получить блокировку в БД, и задание выполняется только репликой, получившей её.
Рассмотрим пример конфигурации Quartz.NET с использованием нескольких реплик и БД в качестве хранилища.
```csharp
public void ConfigureServices(IServiceCollection services)
{
	var connectionString = Configuration.GetConnectionString("DefaultConnection");
	
	services.AddQuartz(q =>
	{
		q.SchedulerId = "AUTO"; //Уникальный Id планировщика назначается автоматически
		q.UseMicrosoftDependencyInjectionScopedJobFactory();
		q.UsePersistentStore(s => //активируем долговременное хранилище в БД
		{
			s.UseSqlServer(connectionString); //Используем SQL Server
			s.UseClustering(); //активируем кластеризацию
			s.UseProperties = true;
			s.UseJsonSerializer();
		});
		//аналогично предыдущему разделу
		var jobKey = new JobKey("Update exchange rates");
		q.AddJob<UpdateExchangeRatesJob>(opts => opts.WithIdentity(jobKey));
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
В этой конфигурации мы настроили хранение списка заданий и триггеров в БД, а также включили использование блокировки БД для обеспечения запуска только одного экземпляра задания.
> [!note] Примечание
> SQLite не поддерживает примитивы блокировки БД, поэтому эту БД нельзя использовать при кластеризации. Однако её по прежнему можно использовать в качестве долговременного хранилища.

Quartz.NET не создаёт необходимые для работы таблицы в БД. На GitHub можно найти [скрипты](https://github.com/quartznet/quartznet/tree/main/database/tables) для создания таблиц в различных БД.
> [!tip] Совет
> Если в вашем проекте используются миграции EF Core для управления БД, рекомендуется использовать их и для создания таблиц для Quartz.NET.

При использовании кластеризации Quartz.NET нужно обязательно некоторые важные вещи, упомянутые на [этой странице документации](https://www.quartz-scheduler.net/documentation/quartz-3.x/tutorial/advanced-enterprise-features.html).

