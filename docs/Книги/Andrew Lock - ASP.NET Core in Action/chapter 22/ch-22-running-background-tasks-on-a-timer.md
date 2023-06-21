---
share: true
tags:
 - NET/background-task
---
# Запуск фоновых задач по таймеру
Рассмотрим, как создать фоновую задачу, которая периодически запускается по таймеру на протяжении всего жизненного цикла приложения.
Для примера возьмем [[ch-21-using-typed-clients-to-encapsulate-http-calls|типизированного клиента]] из прошлого раздела и добавим кеширование значений обменного курса на какое-то время.
Идея заключается в том, что API берёт значения из кэша, обновляемого в фоновом режиме.
> [!note] Примечание
> Альтернативным подходом будет добавление кеширования в сам `ExchangeRatesClient`, минусом такого подхода будет тот факт, что при необходимости обновления курсов придётся выполнить запрос немедленно, что замедлит общий ответ.

Реализуем фоновую задачу с помощью `IHostedService`. Вот он:
```csharp
public interface IHostedService
{
	Task StartAsync(CancellationToken cancellationToken);
	Task StopAsync(CancellationToken cancellationToken);
}
```
Метод `StartAsync`, хоть и является асинхронным, выполняется встроенно как часть процесса запуска приложения.
Фоновые задачи, которые будут выполняться в течение всего жизненного цикла приложения, должны быть запланированны для выполнения в другом потоке, сразу же вернув `Task`.
Для упрощения создания фоновых задач ASP.NET Core предоставляет базовый класс `BackgroundService`, реализующий интерфейс `IHostedService`. Для создания фоновой задачи нужно переопределить метод `ExecuteAsync()`.
Напишем фоновый сервис, извлекающий текущие процентные ставки при помощи типизированного клиента и обновляющий кэш.
```csharp
public class ExchangeRatesHostedService: BackgroundService
{
	private readonly IServiceProvider _provider;
	private readonly ExchangeRatesCache _cache;
	public ExchangeRatesHostedService(
		IServiceProvider provider, ExchangeRatesCache cache)
	{
		_provider = provider;
		_cache = cache;
	}
	
	protected override async Task ExecuteAsync(
		CancellationToken stoppingToken)
	{
		while(!stoppingToken.IsCancellationRequested)
		{
			var client = _provider
				.GetRequiredService<ExchangeRatesClient>()
			var rates = await client.GetLatestRatesAsync();
			_cache.SetRates(rates);
			await Task.Delay(TimeSpan.FromMinutes(5), stoppingToken);
		}
	}
}
```
`ExchangeRatesCache` — это простой объект одиночка, в котором хранятся последние курсы валют. Он должен быть потокобезопасным, так как контроллеры API будут обращаться к нему одновременно.
Остаётся зарегистрировать фоновый сервис в [[di-container|контейнере зависимостей]]:
```csharp
services.AddHttpClient<ExchangeRatesClient>();
services.AddSingleton<ExchangeRatesCache>();

services.AddHostedService<ExchangeRatesHostedService>();
```
В нашем примере для получения типизированного клиента используется *локатор сервисов*. Сделано это потому, что не следует внедрять типизированные клиенты напрямую в фоновые сервисы, так как они создаются синглтонами.
> [!tip] Совет
> Можно избежать использования локатора сервисов, используя [[factory-pattern|паттерн Фабрика]], описанный [здесь](https://www.stevejgordon.co.uk/ihttpclientfactory-patterns-using-typed-clients-from-singleton-services)