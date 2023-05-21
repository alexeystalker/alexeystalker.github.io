---
share: true
tags:
 - NET/authorization
 - security
---
# Предотвращение доступа анонимных пользователей к вашему приложению
Самый простой уровень [[authorization|авторизации]] — позволять только проверенным пользователям выполнять конечную точку. Тут есть два исхода:
- *Пользователь прошел [[authentication|аутентификацию]]* — действие выполняется в обычном режиме;
- *пользователь не прошел аутентификацию* — пользователь не может выполнить конечную точку.

Такой уровень авторизации [[ch-13-authorization-filters|достигается]] при помощи атрибута `[Authorize]`:
```csharp
public class RecipeApiController : ControllerBase
{
	public IActionResult List() // Это действие может выполнить кто угодно
	{
		return Ok();
	}
	
	[Authorize] // Применяем атрибут к действиям, целым контроллерам или страницам
	public IActionResult View() // Это действие могут выполнять только прошедшие аутентификацию пользователи
	{
		return Ok();
	}
}
```
Атрибут `[Authorize]` можно применять в области [[action-method|метода действия]], контроллера, страницы Razor или глобально[^1].

Если применять атрибут глобально, понадобится способ добавить исключения. Это можно сделать, используя атрибут `[AllowAnonymous]`:
```csharp
[Authorize] //Применяем атрибут на весь контроллер
public class AccountController : ControllerBase
{
	public IActionResult ManageAccount()
	{
		return Ok();
	}
	
	[AllowAnonymous]
	public IActionResult Login()
	{
		return Ok();
	}
}
```

> [!Warning] Внимание!
> Если атрибут `[Authorize]` применяется глобально, необходимо добавить атрибут `[AllowAnonymous]` к действиям входа, ошибок и сброса пароля, а также любым другим действиям, которые должны выполнять пользователи, не прошедшие аутентификацию.

[^1]: О глобальном применении атрибута `[Authorize]` можно почитать [в блоге автора](https://andrewlock.net/setting-global-authorization-policies-using-the-defaultpolicy-and-the-fallbackpolicy-in-aspnet-core-3/)