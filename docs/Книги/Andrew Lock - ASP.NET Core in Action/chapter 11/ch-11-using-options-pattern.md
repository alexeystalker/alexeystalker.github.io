---
share: true
tags:
 - NET/configuration
---
# Использование строго типизированных настроек с паттерном Options
Вместо того, чтобы использовать значения по ключу, в виде `Configuration["key"]`, можно использовать строго типизированные настройки. Это наиболее предпочтительный способ.
Рассмотрим пример файла настроек, разделенных на два разных объекта с ключами `"MapSettings"` и `"AppDisplaySettings"`:
```json
{
	"MapSettings": {
		"DefaultZoomLevel": 6,
		"DefaultLocation": {
			"latitude": 50.500,
			"longitude": -4.000
		}
	},
	"AppDisplaySettings": {
		"Title": "Acme Store Locator",
		"ShowCopyright": true
	}
}
```
Тогда при использовании этих настроек можно использовать строго типизарованные объекты:
```csharp
public class IndexModel: PageModel
{
	public IndexModel(IOptions<AppDisplaySettings> options)
	{
		AppDisplaySettings settings = options.Value;
		var title = settings.Title;
		bool showCopyright = settings.ShowCopyright;
	}
}
```
Система ASP.NET Core использует *связыватель*, принимающий коллекцию значений конфигурации и привязывающий их к строго типизированному объекту — классу *options*. Это похоже на концепцию [[model-binding|привязки модели]].
#### [[ch-11-introducing-ioptions-interface|Знакомство с интерфейсом IOptions]]
#### [[ch-11-reloading-with-ioptionssnapshot|Перезагрузка строго типизированных параметров с помощью IOptionsSnapshot]]
#### [[ch-11-designing-options-classes|Разработка классов параметров для автоматической привязки]]
#### [[ch-11-binding-without-ioptions|Связывание строго типизированных настроек без интерфейса IOptions]]