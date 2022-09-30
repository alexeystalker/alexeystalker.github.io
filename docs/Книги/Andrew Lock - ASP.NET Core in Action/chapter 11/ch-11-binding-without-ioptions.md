---
share: true
tags:
 - NET/configuration
---
# Связывание строго типизированных настроек без интерфейса IOptions
Помимо всего прочего, есть возможность привязывать строго типизированные объекты настроек без использования `IOptions`. Например:
```csharp
public IConfiguration Configuration { get; }
public void ConfigureServices(IServiceCollection services)
{
	var settings = new MapSettings();
	Configuration.GetSection("MapSettings").Bind(settings);
}
```
Теперь `MapSettings` может быть внедрен напрямую, без использования `IOptions<>`.
Необходимо учесть, что в этом случае продвинутые функции вроде горячей перезагрузки настроек будут недоступны.
