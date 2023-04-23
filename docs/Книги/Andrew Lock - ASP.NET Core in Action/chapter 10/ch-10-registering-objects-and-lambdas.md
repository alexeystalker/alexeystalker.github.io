---
share: true
tags:
 - NET/ASPNETCore
 - NET/DI
---
# Регистрация сервисов с использованием объектов и лямбда-функций
Очень часто при регистрации сервиса необходимо передать ему какие-то параметры (либо указать конкретный конструктор), или использовать класс, не удовлетворяющий условиям, описанным в [[ch-10-registering-own-services|предыдущем разделе]].
Для этого можно самим создать необходимый объект и зарегистрировать его напрямую. Например:
```csharp
public void ConfigureServices(IServiceCollection services)
{
	...
	services.AddSingleton(
		new EmailServerSettings(
			host: "smtp.server.com",
			port: 25
		));
}
```
Однако в этом случае этот объект будет использоваться всегда.
Чтобы создавать новый объект каждый раз, когда это необходимо, нужно предоставить контейнеру *функцию*, которую контейнер будет использовать.
```csharp
public void ConfigureServices(IServiceCollection services)
{
	...
	services.AddSсoped(provider =>
		new EmailServerSettings(
			host: "smtp.server.com",
			port: 25
		));
}
```
При использовании в функцию передается экземпляр `IServiceProvider`, который можно использовать для получения зарегистрированных сервисов из контейнера через `getService()`.
## Открытые обобщенные типы и внедрение зависимостей
В контейнере можно также зарегистрировать *открытые обобщенные типы*. Обобщенный тип (generic) — это тип вида `Repository<T>`. Можно зарегистрировать реализацию обобщения `IRepository<T>` для любого допустимого `T`:
```csharp
services.AddScoped(typeof(IRepository<>), typeof(Repository<>));
```
Тем самым, когда конструктору потребуется тип `IRepository<T>`, контейнер внедрит `Repository<T>`.
## Группировка регистраций в метод расширения
Чтобы код выглядел более аккуратно, а также чтобы инкапсулировать регистрации отдельных сервисов, можно создать свой метод расширения `Add*`:
```csharp
public static class EmailSenderServiceCollectionExtensions
{
	public static IServiceCollection AddEmailSender(this IServiceCollection services)
	{
		services.AddScoped<IEmailSender, EmailSender>();
		services.AddScoped<NetworkClient>();
		services.AddSingleton<MessageFactory>();
		services.AddSingleton(
			new EmailServerSettings(
				host: "smtp.server.com",
				port: 25
			));
		
		return services;
	}
}
```