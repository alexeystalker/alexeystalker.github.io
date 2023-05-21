---
share: true
tags:
 - NET/configuration
---
# Создание объекта конфигурации для вашего приложения
[[ch-11-configuring-with-createdefaultbuilder|Ранее]] мы видели, как можно использовать метод `CreateDefaultBuilder()` для создания экземпляра `IHostBuilder`. Он отвечает за настройку многих аспектов приложения, включая систему конфигурации в методе `ConfigureAppConfiguration()`, которому передается экземпляр `ConfigurationBuilder`, использующийся для определения конфигурации приложения.
Модель конфигурации ASP.NET Core основана на двух основных конструкциях — `ConfigurationBuiler` (описывает, как построить окончательное представление конфигурации), и `IConfiguration` (содержит сами значения конфигурации).
Настройка конфигурации описывается путём добавления ряда объектов `IConfigurationProvider` в `ConfigurationBuilder` в методе `ConfigureAppConfiguration()`. `IConfigurationProvider` описывают, как загрузить пары "ключ-значение" из определенного источника; метод `Build()` запрашивает у каждого из поставщиков значения, чтобы создать `IConfigurationRoot()`.
ASP.NET Core поставляется с поставщиками конфигурации из следующих местоположений:
- файлы JSON;
- XML-файлы;
- переменные окружения;
- аргументы командной строки;
- файлы INI.

Также, разумеется, можно написать свой поставщик.
Во многих случаях поставщика по умолчанию будет достаточно; он загружает конфигурацию из файла appsettings.json.
Хорошей идеей будет создать “пространство имен” для собственных настроек, создав корневой объект (здесь — `MapSettings`):
```json
{
	"Logging": {
		"LogLevel": {
			"Default": "Information",
			"Microsoft": "Warning",
			"Microsoft.Hosting.Lifetime": "Information"
		}
	},
	"AllowedHosts": "*",
	"MapSettings": {
		"DefaultZoomLevel": 9,
		"DefaultLocation": {
			"latitude": 50.500,
			"longitude": -4.000
		}
	}
}
```
#### [[ch-11-adding-configuration-provider-in-program-cs|Добавление поставщика конфигурации в файле Program.cs]]
#### [[ch-11-using-multiple-providers|Использование нескольких поставщиков для переопределения значений конфигурации]]
#### [[ch-11-storing-config-secrets-safely|Безопасное хранение секретов конфигурации]]
#### [[ch-11-reloading-configuration-values|Перезагрузка значений конфигурации при их изменении]]