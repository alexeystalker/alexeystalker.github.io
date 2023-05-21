---
share: true
tags:
 - NET/ASPNETCore
 - NET/DI
 - NET/MVC
 - NET/Razor
---
# Внедрение сервисов в методы действий, обработчики страниц и представления
[[di-container|Контейнер внедрения зависимостей]] ASP.NET Core поддерживает только внедрение через конструктор, однако есть еще три места, где можно использовать [[dependency-injection-pattern|внедрение зависимостей]]:
- [[action-method|методы действия]]
- [[page-handler|обработчики страницы]]
- шаблоны представлений

## Внедрение сервисов непосредственно в методы действий и обработчики страниц с помощью атрибута \[FromServices\]
Внедрение через конструктор означает, что все зависимости создаются при создании контроллера, независимо от того, будут ли они использованы в методах действий или нет. Если известно, что создание зависимости — затратная операция, то можно внедрить зависимость напрямую в метод. Это делается передачей сервиса в качестве параметра, помеченного атрибутом `[FromServices]`:
```csharp
public class UserController : ControllerBase
{
	[HttpPost("register")]
	public IActionResult RegisterUser([FromServices]IMessageSender messageSender, string username)
	{
		messageSender.SendMessage(username);
		return Ok();
	}
	[HttpPost("promote")]
	public IActionResult PromoteUser([FromServices] IPromotionService promoService, string username, int level)
	{
		promoService.PromoteUser(username, level);
	}
}
```
В Razor Pages также можно использовать эту возможность:
```csharp
public class PromoteUserModel : PageModel
{
	public void OnGet() {}
	public IActionResult OnPost(
		[FromServices] IPromotionService promoService,
		string username, int level)
	{
		promoService.PromoteUser(username, level);
		return RedirectToPage("success");
	}
}
```

## Внедрение сервисов в шаблоны представлений
Представим, что у нас есть простой сервис HtmlGenerator, помогающий сгенерировать HTML-код в шаблонах представления. Вопрос — как передать зарегистрированный в контейнере сервис шаблонам.
Один из вариантов — внедрить сервис в `PageModel` через конструктор и представить его как свойство.
Другой вариант — использовать директиву `@inject`:
```razor
@inject HtmlGenerator htmlHelper
<h1>The page title</h1>
<footer>@htmlHelper.Copyright()</footer>
```