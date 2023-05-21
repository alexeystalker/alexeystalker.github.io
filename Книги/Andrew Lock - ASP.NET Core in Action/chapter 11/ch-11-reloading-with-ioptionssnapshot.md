---
share: true
tags:
 - NET/configuration
---
# Перезагрузка строго типизированных параметров с помощью IOptionsSnapshot
`IOptions<T>` — отличный способ для обеспечения строго типизированного доступа к настройкам. Проблема в том, что в этом случае значения параметров не меняются при изменении значений источника, например, файла appsettings.json.
Если необходимо получать измененные настройки без перезапуска приложения, можно использовать интерфейс `IOptionsSnapshot<T>` — его свойство `Value` обновляется, если базовая конфигурация изменилась.
Использование `IOptionsSnapshot<T>` не отличается от `IOptions<T>`:
```csharp
public class IndexModel : PageModel
{
	public IndexModel(IOptionsSnapshot<AppDisplaySettings> options)
	{
		AppDisplaySettings settings = options.Value;
		var title = settings.Title;
	}
}
```
