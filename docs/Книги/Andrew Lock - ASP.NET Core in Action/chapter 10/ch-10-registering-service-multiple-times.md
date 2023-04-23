---
share: true
tags:
 - NET/ASPNETCore
 - NET/DI
---
# Многократная регистрация сервиса в контейнере
Часто у одного интерфейса появляется несколько реализаций. Например, интерфейс
```csharp
public interface IMessageSender
{
	public void SendMessage(string message);
}
```
может иметь несколько реализаций: `EmailSender`, `SmsSender` и `FacebookSender`.
Есть несколько способов зарегистрировать эти реализации в [[di-container|контейнере]] и внедрить их в `UserController`, в зависимости от сценария.
## Внедрение нескольких реализаций интерфейса
Пусть мы хотим отправить сообщение, используя каждую из реализаций `IMessageSender`. Самый простой способ для этого — зарегистрировать все реализации в контейнере, а затем использовать цикл `foreach`.
Регистрация:
```csharp
public void ConfigureServices(IServiceCollection services)
{
	services.AddControllers();
	services.AddScoped<IMessageSender, EmailSender>();
	services.AddScoped<IMessageSender, SmsSender>();
	services.AddScoped<IMessageSender, FacebookSender>();
}
```
Теперь можно внедрить  `IEnumerable<IMessageSender>` в `UserController` и использовать `foreach`:
```csharp
public class UserController : ControllerBase
{
	private readonly IEnumerable<IMessageSender> _messageSenders;
	public UserController(IEnumerable<IMessageSender> messageSenders)
	{
		_messageSenders = messageSenders;
	}
	[HttpPost]
	public IActionResult RegisterUser(string username)
	{
		foreach (var messageSender in _messageSenders)
		{
			messageSender.SendMessage(username);
		}
		return Ok();
	}
}
```

## Внедрение одной реализации при регистрации нескольких сервисов
В случае, когда зарегистрированно несколько реализаций одного интерфейса, и сервис требует только одну реализацию, как зависимость, то есть
```csharp
public class SingleMessageSender
{
	private readonly IMessageSender _messageSender;
}
```
контейнер использует реализацию, зарегистрированную *последней*.
Обычно это бывает полезно для замены встроенных регистраций собственными сервисами.
Основным недостатком такого подхода будет то, что внедрение `IEnumerable<T>` всё еще доступно.
## Условная регистрация сервисов с помощью TryAdd
В пространстве имён `Microsoft.Extensions.DependencyInjection.Extensions` можно найти методы расширения для условной регистрации, например `TryAddScoped`. Он проверяет, не зарегистрирован ли сервис в контейнере, перед тем, как вызвать `AddScoped` для реализации. Обычно этот способ применяют при написании библиотек для нескольких приложений.
Также можно *заменить* ранее зарегистрированную реализацию при помощи метода `Replace()`:
```csharp
services.Replace(new ServiceDescriptor(typeof(IMessageSender), typeof(SmsSender), ServiceLifetime.Scoped));
```
Важно - при использовании `Replace()` нужно указать тот же жизненный цикл, что и у заменяемого сервиса.