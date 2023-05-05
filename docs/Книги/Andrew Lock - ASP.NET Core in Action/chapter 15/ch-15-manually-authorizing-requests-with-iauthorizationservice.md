---
share: true
tags:
 - NET/authorization
 - security
---
# Ручная авторизация запросов с помощью интерфейса IAuthorizationService
Здесь нам придётся отказаться от *декларативного* подхода в пользу *императивного*.
ASP.NET Core предоставляет интерфейс IAuthorizationService, который можно внедрить через DI в Razor Pages и контроллеры для императивной авторизации:
```csharp
[Authorize]
public class EditModel : PageModel
{
	[BindProperty]
	public Recipe Recipe { get; set; }
	
	private readonly RecipeService _service;
	private readonly IAuthorizationService _authService;
	
	public EditModel(
		RecipeService recipeSerivce,
		IAuthorizationService authService)
	{
		_service = service;
		_authService = authService;
	}
	
	public async Task<IActionResult> OnGet(int id)
	{
		Recipe = _service.GetRecipe(id);
		var authResult = await _authService
			.AuthorizeAsync(User, Recipe, "CanManageRecipe");
		if (!authResult.Succeeded)
		{
			return new ForbidResult();
		}
	}
}
```
`IAuthorizationService` предоставляет метод `AuthorizeAsync`, которому требуются три вещи:
- `ClaimsPrincipal`, в `PageModel` это свойство `User`;
- авторизуемый ресурс `Recipe`;
- политика — `"CanManageRecipe"`.

Попытка авторизации возвращает объект `AuthorizationResult`, успешность определяется свойством `Succeeded`.
Также необходимо определить политику `"CanManageRecipe"`. Этот процесс совпадает с [[ch-15-creating-a-policy-with-custom-requirement-and-handler|таковым]] для декларативного подхода.
Создаем требование:
```csharp
public class IsRecipeOwnerRequirement : IAuthorizationRequirement { }
```
Добавляем политику в `ConfigureServices`:
```csharp
public void ConfigureServices(IServiceCollection services)
{
	...
	services.AddAuthorization(options => {
		options.AddPolicy("CanManageRecipe", policyBuilder =>
			policyBuilder.AddRequirements(new IsRecipeOwnerRequirement()));
	});
	...
}
```
Далее надо [[ch-15-creating-resource-based-authorization-handler|создать обработчик]].