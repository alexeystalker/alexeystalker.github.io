---
share: true
tags:
 - NET/configuration
---
# Использование сервисов для настройки IOptions с помощью IConfigurationOptions
Распространённой и рекомендуемой практикой является [[ch-11-using-options-pattern|привязка]] объекта конфигурации к `IOptions<T>`. Обычно такая привязка настраивается в `Startup.ConfigureServices()` путём вызова `services.Configure<T>()`, то есть
```csharp
public void ConfigureServices(IServiceCollection services)
{
	services.Configure<CurrencyOptions>(
		Configuration.GetSection("Currencies"));
}
```
У метода `services.Configure<T>()` есть перегруженный вариант, позволяющий предоставить лямбда-функцию, которую фреймворк использует для настройки объекта `CurrencyOptions`.
Например, если мы хотим задать свойство `Currencies` в сконфигурированном ранее объекте:
```csharp
public void ConfigureServices(IServiceCollection services)
{
	services.Configure<CurrencyOptions>(
		Configuration.GetSection("Currencies"));
	services.Configure<CurrencyOptions>(options =>
		options.Currencies => new string[] { "GBR", "USD" });
}
```
Каждый вызов `Configure<T>()` добавляет еще шаг конфигурации к объекту `CurrencyOptions`. Когда [[di-container|контейнеру внедрения зависимостей]] требуется экземпляр `IOptions<CurrencyOptions>`, каждый из шагов выполняется по очереди.
![[Pasted image 20220917152620.png]]
Теперь предположим, что мы можем получить значения для подстановки в лямбда-функцию только на этапе, когда [[di-container|контейнер зависимостей]] полностью сконфигурирован? Например, из БД или какого-либо удалённого сервиса.
Решение состоит в том, чтобы отложить настройку объекта `IOptions<T>` до последнего момента, непосредственно перед тем, как контейнер должен будет создать его. ASP.NET Core предоставляет механизм с интерфейсом `IConfigureOptions<T>`, который реализуется в классе конфигурации. Этот класс может использовать [[dependency-injection-pattern|внедрение зависимостей]]:
```csharp
public class ConfigureCurrencyOptions: IConfigureOptions<CurrencyOptions>
{
	private readonly ICurrencyProvider _currencyProvider;
	public ConfigureCurrencyOptions(ICurrencyProvider currencyProvider)
	{
		_currencyProvider = currencyProvider;
	}
	
	public void Configure(CurrencyOptions options)
	{
		options.Currencies = _currencyProvider.GetCurrencies();
	}
}
```
Теперь зарегистрируем этот класс в контейнере внедрения зависимостей. Так как тут важен порядок, то если мы хотим, чтобы `ConfigureCurrencyOptions` выполнялся *после* привязки к конфигурации, нужно добавить его после вызова `services.Configure<T>()`:
```csharp
public void ConfigureServices(IServiceCollection services)
{
	services.Configure<CurrencyOptions>(
		Configuration.GetSection("Currencies"));
	services.AddSingleton
		<IConfigureOptions<CurrencyOptions>, ConfigureCurrencyOptions>();
}
```
> [!attention] Внимание!
> Объект `ConfigureCurrencyOptions` регистрируется как Singleton, поэтому он будет захватывать сервисы с типом жизненного цикла Scoped или Transient[^1].

[^1]: Если добавляется сервис с временем жизни Scoped (например `DbContext`) то нужно проследить, что память правильно освобождена. Автор описывает, что нужно делать, в [своём блоге](https://andrewlock.net/access-services-inside-options-and-startup-using-configureoptions/)