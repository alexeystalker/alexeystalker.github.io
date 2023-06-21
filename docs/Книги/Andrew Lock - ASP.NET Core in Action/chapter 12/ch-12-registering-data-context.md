---
share: true
tags:
 - NET/EFCore
---
# Регистрация контекста данных
Для использования контекста данных в приложении его надо зарегистрировать в [[di-container|контейнере зависимостей]]. Также при регистрации происходит конфигурирование провайдера.
EF Core предоставляет обобщенный метод `AddDbContext<T>` для регистрации контекста в `ConfigureServices()`.
Типичный пример конфигурации при использовании MS SQL Server:
```csharp
public void ConfigureServices(IServiceCollection services)
{
	var connString = Configuration.GetConnectionString("DefaultConnection");
	services.AddDbContext<AppDbContext>(
		options => options.UseSqlServer(connString));
}
```
Для провайдера другой БД необходимо использовать соответствующий метод `Use*`.
Строка подключения является типичным [[secret-setting|секретом]], поэтому наиболее правильно получать ее из [[ch-11-storing-config-secrets-safely|конфигурации]].
