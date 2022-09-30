---
share: true
tags:
 - NET/configuration
---
# Знакомство с интефейсом IOptions
Пусть мы хотим связать раздел настроек `AppDisplaySettings`из [[ch-11-using-options-pattern|этого раздела]] с POCO классом. Тогда он будет иметь такой вид:
```csharp
public class AppDisplaySettings
{
	public string Title { get; set; }
	public bool ShowCopyright { get; set; }
}
```
Чтобы быть успешно связанным, класс параметров не должен быть абстрактным, а также должен иметь открытый (public) конструктор без параметров. Связыватель установит все открытые свойства, совпадающие со значениями конфигурации. Свойства могут быть не только примитивных типов.
ASP.NET Core предоставляет интерфейс `IOptions<T>` с одним свойством Value, которое содержит объект класса со значениями параметров. Классы параметров настраиваются в секции `ConfigureServices` класса `Startup`:
```csharp
public IConfiguration Configuration { get; }
public void ConfigureServices(IServiceCollection services)
{
	services.Configure<MapSettings>(
		Configuration.GetSection("MapSettings"));
	services.Configure<AppDisplaySettings>(
		Configuration.GetSection("AppDisplaySettings"));
}
```
Каждый вызов `Configure<T>` выполняет следующее:
1. Создает экземпляр `ConfigureOptions<T>`, который указывает, что `IOptions<T>` должен быть настроен на основе конфигурации. Если `Configure<T>` вызывается несколько раз, будет создано несколько объектов `ConfigureOptions<T>`;
2. Каждый экземпляр `ConfigureOptions<T>` привязывает секцию `IConfiguration` к экземпляру POCO класса `T`. Все открытые свойства этого класса задаются на основе ключей из предоставленной `ConfigurationSection`;
3. Интерфейс `IOptions<T>` регистрируется в [[di-container|контейнере зависимостей]] как синглтон, с привязанным объектом POCO в свойстве `Value`.

Если `Configure<T>` не будет вызван, при внедрении `IOptions<T>` объект класса параметров будет иметь значения свойств по умолчанию.

Если требуется, чтобы конфигурация поддерживала обновление значений параметров "на горячую", вместо `IOptions<T>` [[ch-11-reloading-with-ioptionssnapshot|нужно использовать]] `IOptionsSnapshot<T>`.