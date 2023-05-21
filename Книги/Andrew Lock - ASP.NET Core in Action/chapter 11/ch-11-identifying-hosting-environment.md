---
share: true
tags:
 - NET/configuration
 - NET/environment
---
# Определение окружения размещения
Как уже было [[ch-11-configuring-with-createdefaultbuilder|показано]], метод `ConfiguringHostConfiguration` — это место, где определяется, как приложение вычисляет окружение размещения. По умолчанию (`CreateDefaultBuilder()`) используется переменная окружения. `HostBuilder` ищет переменную окружения `ASPNETCORE_ENVIRONMENT` и использует ее для создания объекта `IHostEnvironment`. Также можно использовать переменную `DOTNET_ENVIRONMENT`, но `ASPNETCORE_ENVIRONMENT` имеет приоритет.
Интерфейс `IHostEnvironment` предоставляет ряд полезных свойств, например, `ContentRootPath`, указывающий на папку с файлами контента приложения.
Нас интересует свойство `EnvironmentName`. Ему присваивается значение переменной `ASPNETCORE_ENVIRONMENT`, поэтому строка может быть любой. Однако, в большинстве случаев нужно придерживаться трёх значений:
- `"Development"`;
- `"Staging"`;
- `"Production"`.

Существует несколько методов расширения для работы с этими тремя значениями:
- `IHostEnvironment.IsDevelopment()`;
- `IHostEnvironment.IsStaging()`;
- `IHostEnvironment.IsProduction()`;
- `IhostEnvironment.IsEnvironment(string environmentName)`.

[[ch-8-environment-tag-helper|Вот пример]], в котором используется значение, предоставляемое `IHostEnvironment`.
