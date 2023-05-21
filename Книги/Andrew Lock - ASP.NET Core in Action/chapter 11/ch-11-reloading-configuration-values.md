---
share: true
tags:
 - NET/configuration
---
# Перезагрузка значений конфигурации при их изменении
Существует масса сценариев, когда требуется менять значения настроек без перезапуска приложения.
Для поддержки такого сценария у методов `Add*File` есть перегруженные варианты с параметром `reloadOnChange`. При значении `true` для этого параметра приложение бедет отслеживать файловую систему для поиска изменений в файле и при необходимости пересобирёт `IConfiguration`:
```csharp
public class Program
{
	/*дополнительная конфигурация*/
	public static void AddAppConfiguration(
		HostBuilderContext hostingContext,
		IConfigurationBuilder config)
	{
		config.AddJsonFile(
			"appsettings.json",
			optional: true,
			reloadOnChange: true);
	}
}
```
