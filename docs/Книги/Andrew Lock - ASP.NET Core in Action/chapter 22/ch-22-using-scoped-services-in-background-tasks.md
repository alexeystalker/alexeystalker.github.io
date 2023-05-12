---
share: true
tags:
 - NET/background-task
---
# Использование сервисов с жизненным циклом Scope в фоновых задачах
Фоновые сервисы, реализующие `IHostedService`, создаются один раз при запуске приложения, то есть, фактически являются синглтонами. Как быть, если нужно использовать сервисы с жизненным циклом Scoped?
Разберем следующий пример. Немного изменим пример с кешированием из [[ch-22-running-background-tasks-on-a-timer|предыдущего раздела]], и станем хранить курсы в БД.
`DbContext` от EF Core регистрируется с жизненным циклом Scoped, что означает необходимость получать его из `ExchangeRatesHostedService`, но не через внедрение в конструкторе.
В типичном ASP.NET Core приложении фреймворк создаёт новую область (Scope) контейнера при получении запроса, перед выполнением конвейера промежуточного ПО. Однако в фоновом сервисе нет запросов, поэтому новые области не создаются. Поэтому надо создавать новую область самим. Для этого используется метод `IServiceProvider.CreateScope()`.
Получаемая таким образом область `IServiceScope` реализует интерфейс `IDisposable`.
> [!attention] Внимание!
> Не забывайте избавляться (Dispose) от области `IServiceScope` после окончания использования — так вы предотвратите утечку памяти. Лучше всего использовать инструкцию `using`.

В следующем примере перепишем `ExchangeRatesHostedService`, создавая новую область для каждой итерации цикла `while`.
```csharp
public class ExchangeRatesHostedService: BackgroundService
{
	private readonly IServiceProvider _provider;
	public ExchangeRatesHostedService(IServiceProvider provider)
	{
		_provider = provider;
	}
	
	protected override async Task ExecuteAsync(CancellationToken token)
	{
		while (!token.IsCancellationRequested)
		{
			using(IServiceScope scope = _provider.CreateScope())
			{
				var scopedProvider = scope.ServiceProvider;
				var client = scopedProvider
					.GetRequiredService<ExchangeRatesClient>();
				var context = scopedProvider
					.GetRequiredService<AppDbContext>();
				
				var rates = await client.GetLatestRatesAsync();
				
				context.Add(rates);
				await context.SaveChanges(rates);
			}
			await Task.Delay(TimeSpan.FromMinutes(5), token);
		}
	}
}
```
Создание таких областей — общее решение, когда нужен доступ к сервисам с жизненным циклом Scoped вне контекста запроса. Например [[ch-19-using-services-to-configure-ioptions-with-iconfigureoptions|реализация]] интерфейса `IConfigureOptions`.
> [!tip] Совет
> Чтобы сделать использование локатора сервисов менее запутанным, рекомендуется выделять тело задачу в отдельный класс и использовать локацию сервисов только для этого класса, как показано в разделе "Использование службы с заданной областью в фоновой задаче" [этого документа](https://learn.microsoft.com/aspnet/core/fundamentals/host/hosted-services?view=aspnetcore-6.0&tabs=visual-studio#consuming-a-scoped-service-in-a-background-task)

