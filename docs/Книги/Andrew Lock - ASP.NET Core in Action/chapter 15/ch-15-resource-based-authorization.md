---
share: true
tags:
 - NET/authorization
 - security
---
# Управление доступом с авторизацией на основе ресурсов
Авторизация на основе ресурсов — распространённая задача для приложений, особенно когда есть пользователи, которые могут создавать или редактировать какие-то документы.
Рассмотрим [[ch-12-sample-application|наше приложение с рецептами]]. Добавим в него новое поведение:
- только авторизованные пользователи должны иметь возможность создавать новые рецепты;
- вы можете редактировать только созданные вами рецепты.

Первое требование легко выполнить, декорировав страницу Create.cshtml атрибутом `[Authorize]`, как обсуждалось [[ch-15-preventing-anonymous-users-from-accessing-your-application|ранее]].
```csharp
[Authorize]
public class CreateModel : PageModel
{
	[BindProperty]
	public CreateRecipeCommand Input { get; set; }
	
	public OnGet()
	{
		Input = new CreateRecipeCommand();
	}
	
	public async Task<IActionResult> OnPost()
	{
		//Тело метода опустим
	}
}
```

А вот для того, чтобы выполнить второе требование, нужно применить другую технику.
Для начала предположим, мы решили проверять принадлежность рецепта прямо в [[page-handler|обработчике страницы]]:
```csharp
...
public IActionResult OnGet(int id)
{
	var recipe = _service.GetRecipe(id);
	var createdById = recipe.CreatedById;
	//Авторизация пользователя на основе createdById
	if(isAuthorized)
	{
		return View(recipe);
	}
	...
}
...
```
#### [[ch-15-manually-authorizing-requests-with-iauthorizationservice|Ручная авторизация запросов с помощью интерфейса IAuthorizationService]]
#### [[ch-15-creating-resource-based-authorization-handler|Создание обработчика AuthorizationHandler на основе ресурсов]]


%% Всё, описанное в этой карточке (и вложенных), работает И для Razor Pages, И для WebApi. Однако, для WebApi/контроллеров MVC можно сделать кастомные атрибуты, как у нас в Компасе. Надо не забыть вычленить логику и записать в отдельные карточки %%