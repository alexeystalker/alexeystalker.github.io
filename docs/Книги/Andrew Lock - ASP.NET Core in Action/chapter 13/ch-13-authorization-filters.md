---
share: true
tags:
 - NET/ASPNETCore/filters
---
# Фильтры авторизации: защита API
[[ch-14-authentication|Аутентификация]] и [[ch-15-authorization|авторизация]] — взаимосвязанные фундаментальные концепции безопасности.

[[authentication|Аутентификация]] — определение того, *кто* сделал запрос. [[authorization|Авторизация]] — определение, *к чему* пользователь имеет доступ.

Фильтры авторизации запускаются в [[ch-13-mvc-filter-pipeline|конвейере]] фильтров первыми перед всеми остальными фильтрами.
Можно написать собственные фильтры авторизации, реализовав `IAuthorizationFilter` или `IAsyncAuthorizationFilter`. Однако, автор не рекомендует этого делать (???).
В основ фреймворка авторизации лежит фильтр авторизации `AuthorizeFilter`, который можно добавить, декорировав действия или контроллер атрибутом `[Authorize]`
```csharp
public class RecipeApiController : ControllerBase
{
	public IActionResult Get(int id)
	{
		// Тело метода
	}
	[Authorize]
	public IActionResult Edit(int id, [FromBody] UpdateRecipeCommand command)
	{
		// Тело метода
	}
}
```
