---
share: true
tags:
 - NET/DI
---
# Создание слабосвязанного кода
[[coupling|Связанность]] — важная концепция ООП. Онп означает, как данный класс зависит от других.
[[ch-10-understanding-benefits-of-di|Пример]] с классами `UserController` и `EmailSender` — образец сильной связанности; объект `EmailSender` создавался напрямую. Кроме того, этот код сложно тестировать — любые попытки выполнить метод приведут к отправке электронного письма.
Использование `EmailSender` в качестве параметра конструктора уменьшило связанность системы, однако, чтобы снизить ее еще больше (а также сделать `UseController` тестируемым), необходимо привязываться не к реализации, а к интерфейсу — это поможет отвязаться от конкретной реализации (и заменять их при необходимости).
![[Pasted image 20220319111636.png]]
В качестве примера можно создать интерфейс `IEmailSender`:
```csharp
public interface IEmailSender
{
	public void SendEmail(string username);
}
```
Тогда `UserController` сможет зависеть от этого интерфейса:
```csharp
public class UserController : ControllerBase
{
	private readonly IEmailSender _emailSender;
	public UserController(IEmailSender emailSender)
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
Теперь можно в любой момент поменять реализацию интерфейса (например, во время тестирования) без необходимости изменять код `UserController`.

Но как приложение узнает, какую реализацию передать в `UserController`? Для этого нужно [[ch-10-registering-own-services|зарегистрировать]] сервис `EmailSender` в [[di-container|контейнере зависимостей]]. Обычно это выглядит как указание "для интерфейса X используйте реализацию Y".