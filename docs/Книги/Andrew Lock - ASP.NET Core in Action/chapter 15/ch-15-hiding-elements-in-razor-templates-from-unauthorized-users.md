---
share: true
tags:
 - NET/Razor
 - NET/authorization
 - security
---
# Скрытие элементов в шаблонах Razor от незарегистрированных пользователей
> [!warning]- Внимание
> Пользовательский интерфейс легко обойти, поэтому важно всегда авторизовать методы действия и страницы Razor на сервере, а не только на стороне клиента.

Для пользовательского опыта было бы лучше, чтоб элементы интерфейса — кнопки или ссылки — выполняющие запрещенные для конкретного пользователя действия, были бы скрыты от этого пользователя.

В качестве примера рассмотрим страницу просмотра рецепта для нашего [[ch-12-sample-application|приложения]]. Добавим возможность скрыть кнопку редактирования.
```csharp
public class ViewModel : PageModel
{
	public Recipe Recipe { get; set; }
	public bool CanEditRecipe { get; set; }
	private readonly RecipeService _service;
	private readonly IAuthorizationService _authService;
	public ViewModel(RecipeService service,
		IAuthorizationService authService)
	{
		_service = service;
		_authService = authService;
	}
	public async Task<IActionResult> OnGetAsync(int id)
	{
		Recipe = _service.GetRecipe(id);
		var isAuthorised = await _authService.AuthorizeAsync(User, Recipe, "CanManageRecipe");
		CanEditRecipe = isAuthorised.Succeeded;
		return Page();
	}
}
```
Здесь мы передаём в свойство `CanEditRecipe` результат валидации.
Затем, добавим условие в шаблон страницы:
```razor
@if(Model.CanEditRecipe)
{
	<a asp-page="Edit" asp-route-id="@Model.Id" class="btn btn-primary">Edit</a>
}
```

> [!warning] Внимание
> Несмотря на то, что кнопка `Edit` теперь скрыта от пользователя, злоумышленник всё равно может перейти на страницу Edit.cshtml, поэтому очень важно сохранить [[ch-15-resource-based-authorization|проверку]] авторизации на этой странице.

