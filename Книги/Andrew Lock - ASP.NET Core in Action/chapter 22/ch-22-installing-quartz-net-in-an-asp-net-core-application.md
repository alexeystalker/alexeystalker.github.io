---
share: true
tags:
 - NET/worker-service
---
# Установка Quartz.NET в приложение ASP.NET Core
Рассмотрим, как установить планировщик Quartz.NET в приложение ASP.NET Core, который будет работать, как реализация `IHostedService`, планируя и выполняя задания.
> [!info] Определение
> *Задание* в Quartz.NET — это выполняемая задача, реализующая интерфейс `IJob`.

Quartz.NET можно установить как в само приложение ASP.NET Core, так и в воркера. Для этого:
1. установим NuGet-пакет Quartz.AspNetCore в проект;
2. добавим необходимые сервисы при помощи метода `AddQuartz`, также по необходимости настроив использование [[di-container|контейнера зависимостей]] для получения заданий;
3. добавим планировщик, вызвав метод `AddQuartzHostedService()`. Также установим параметр `WaitForJobsToComplete = true`;

Вот пример кода:
```csharp
public void ConfigureServices(IServiceCollection services)
{
	services.AddQuartz(q => q.UseMicrosoftDependencyInjectionScopedJobFactory());
	services.AddQuartzHostedService(q => q.WaitForJobsToComplete = true);
}
```
Далее, если мы запустим наше приложение (через `dotnet run` или клавишу **F5** в VS), увидим, что планировщик запустился, но заданий пока нет.
> [!tip] Совет
> Рекомендуется запускать приложение *до* добавления заданий, чтобы убедиться, что всё настроено правильно.

