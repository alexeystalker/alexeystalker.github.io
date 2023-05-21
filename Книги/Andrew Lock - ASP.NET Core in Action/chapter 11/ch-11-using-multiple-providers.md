---
share: true
tags:
 - NET/configuration
---
# Использование нескольких поставщиков для переопределения значений конфигурации
При добавлении нескольких поставщиков конфигурации важен порядок — значения конфигурации от более поздних поставщиков перезапишут значения с тем же ключом от более ранних поставщиков.
```csharp
public class Program
{
	//Также можно было оформить в виде лямбды
	public static void AddAppConfiguration(HostBuilderContext hostingContext, IConfigurationBuilder config)
	{
		config.Sources.Clear();
		config
			.AddJsonFile("sharedsettings.json", optional: true)
			.AddJsonFile("appsettings.json", optional: true)
			.AddEnvironmentVariables();
	}
}
```
Здесь поставщик переменных окружения добавлен последним, и, следовательно, его значения будут иметь приоритет. Такой дизайн особенно полезен при [[ch-11-storing-config-secrets-safely|работе с секретами]].