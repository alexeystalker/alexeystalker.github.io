---
share: true
tags:
 - NET/DI
---
# Преимущества внедрения зависимостей
Рассмотрим пример кода без [[dependency-injection-pattern|внедрения зависимостей]]:
```csharp
public class UserController : ControllerBase
{
	[HttpPost("register")]
	public IActionResult RegisterUser(string username)
	{
		var emailSender = new EmailSender();
		emailSender.SendEmail(username);
		return Ok();
	}
}
```
Предположим, что `EmailSender` для отправки письма должен выполнять следующее:
- формировать электронное письмо (`MessageFactory`);
- сконфигурировать настройки почтового сервера (`EmailServerSettings`);
- отправить письмо на почтовый сервер (`NetworkClient`).

Чтобы соблюсти [[single-responsibility-principle|принцип единственной ответственности]], каждую из этих задач должен выполнять объект отдельного класса. Следовательно, `EmailSender` будет зависеть от этих классов. Образуется [[dependency-graph|граф зависимостей]]:
![[Pasted image 20220318203314.png]]
```csharp
public class EmailSender
{
	private readonly NetworkClient _client;
	private readonly MessageFactory _factory;
	public EmailSender(MessageFactory factory, NetworkClient client)
	{
		_factory = factory;
		_client = client;
	}
	public void SendEmail(string username)
	{
		var email = _factory.Create(username);
		_client.SendEmail(email);
		Console.WriteLine($"Email sent to {username}");
	}
}
```
А `NetworkClient`, в свою очередь, зависит от `EmailServerSettings`:
```csharp
public class NetworkClient
{
	private readonly EmailServerSettings _settings;
	public NetworkClient(EmailServerSettings settings)
	{
		_settings = settings;
	}
	public void SendEmail(Email email)
	{
		//реализация
	}
}
```
Тогда метод `RegisterUser` класса `UserController` выглядит так:
```csharp
public IActionResult RegisterUser(string username)
{
	var emailSender = new EmailSender(
		new MessageFactory(),
		new NetworkClient(
			new EmailServerSettings(
				host: "smtp.server.com",
				port: 25
			)
		)
	);
	emailSender.SendEmail(username);
	return Ok();
}
```
Какие проблемы возникли:
- *несоблюдение принципа единственной ответственности* — наш код отвечает и за создание EmailSender, и за отправку письма;
- *слишком много церемоний* — только две строки кода в методе `RegisterUser` делают что-то полезное;
- *привязка к реализации* — если потребуется изменить зависимости у `EmailSender`, нужно будет обновить все места, где он используется.
- `UserController` *неявно* зависит от `EmailSender`.

[[dependency-injection-pattern|Внедрение зависимостей]] направлено на решение проблемы создания [[dependency-graph|графа зависимостей]] путём *инвертирования* цепочки зависимостей. Вместо того, чтобы создавать зависимости внутри реализации, уже созданный экземпляр явно внедряется через конструктор.
Для осуществления этого нам нужно *что-то* для создания объектов. Эта сущность называется [[di-container|контейнером внедрения зависимостей]].
![[Pasted image 20220318210548.png]]
Посмотрим, как упростится класс `UserController` при использовании внедрения зависимостей:
```csharp
public class UserController : ControllerBase
{
	private readonly EmailSender _emailSender;
	public UserController(EmailSender emailSender)
	{
		_emailSender = emailSender;	
	}
	[HttpPost("register")]
	public IActionResult RegisterUser(string username)
	{
		_emailSender.SendEmail(username);
		return Ok();
	}
}
```
Как видно, все `new` исчезли, код легко понять.