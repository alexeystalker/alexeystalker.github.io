---
share: true
tags:
 - NET/authorization
 - security
---
# Создание обработчика AuthorizationHandler на основе ресурсов
Обработчик на основе ресурсов реализуется путём наследования от `AuthorizationHandler<TRequirement, TResource>` (ср. с обработчиком, в котором указан [[ch-15-creating-a-policy-with-custom-requirement-and-handler|только тип требования]]):
```csharp
public class IsRecipeOwnerHandler : 
		AuthorizationHandler<IsRecipeOwnerRequirement, Recipe>
{
	private readonly UserManager<ApplicationUser> _userManager;
	public IsRecipeOwnerHandler(UserManager<ApplicationUser> userManager)
	{
		_userManager = userManager;
	}
	protected override async Task HandleRequirementAsync(
		AuthorizationHandlerContext context,
		IsRecipeOwnerRequirement requirement,
		Recipe resource)
	{
		var appUser = await _userManager.GetUserAsync(context.User);
		if (appUser == null) //аутентификация не пройдена
		{
			return;
		}
		if (resource.CreatedById == appUser.Id)
		{
			context.Succeed(requirement);
		}
	}
}
```

Здесь используется `UserManager` для загрузки [[ch-14-updating-ef-core-data-model-to-support-identity|сущности]] `ApplicationUser` на основе объекта `ClaimsPrincipal` из запроса.
> [!Important] От меня
> В примере проверка, что пользователь аутентифицирован, отдана на откуп `UserManager`. Если мы не используем ASP.NET Core Identity, а используем что-то своё, то в этом случае нужно проверять `ClaimsPrincipal` напрямую. Например, так:
> ```csharp
> if(!(context.User.Identity?.IsAuthenticated ?? false))
>	return;
> ```

Также мы видим, что, помимо прочего, в метод `HandleRequirementAsync` передается ресурс `Recipe`; это объект, который [[ch-15-manually-authorizing-requests-with-iauthorizationservice|передавали]] в метод `AuthorizeAsync` в `IAuthorizationService`. Используем этот ресурс, чтобы проверить, что он создан текущим пользователем.

Осталось только зарегистрировать обработчик в DI контейнере:
```csharp
services.AddScoped<IAuthorizationHandler, IsRecipeOwnerHandler>();
```
Здесь используется метод `AddScoped()`, так как обработчик использует зависимость `UserManager<>`, которая применяет EF Core.