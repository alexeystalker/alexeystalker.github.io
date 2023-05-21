---
share: true
tags:
 - NET/ASPNETCore/filters
 - NET/MVC
---
# Фильтры действий: настройка привязки модели и результатов действий
Фильтры действий запускаются [[ch-13-mvc-filter-pipeline|после]] [[model-binding|привязки модели]], до выполнения [[action-method|метода действия]]. Благодаря своему расположению эти фильтры получают доступ ко всем аргументам метода. Кроме этого, они запускаются после выполнения метода действия и при желании могут изменить или заменить `IActionResult`, возвращаемый действием.

> [!NOTE]
> фильтры действий не выполняются для Razor Pages

ASP.NET Core включает в себя несколько стандартных фильтров действий, один из которых — `ResponseCacheFilter`, задающий заголовки кэширования для ответов.

Создадим два специальных фильтра для [[ch-13-creating-custom-filters-for-your-application|нашего контроллера]]:
- `ValidateModelAttribute` — возвращает `BadrequestResult`, если состояние модели не является валидным;
- `EnsureRecipeExistsAttribute` — будет проверять, что рецепт существует, используя аргумент `id`.

В случае использования атрибута `[ApiController]` проверка и возврат кода `400 Bad Request` происходит автоматически, однако, если использование этого атрибута невозможно, поможет фильтр `ValidateModelAttribute`:
```csharp
public class ValidateModelAttribute : ActionFilterAttribute
{
	public override void OnActionExecuting(ActionExecutingContext context)
	{
		if (!context.ModelState.IsValid)
		{
			context.Result = new BadRequestObjectResult(context.ModelState);
		}
	}
}
```
Отметим следующее:
- Наследуемся от абстрактного класса `ActionFilterAttribute`. Он реализует интерфейсы `IActionFilter` и `IResultFilter`, а также их асинхронные аналоги[^1], поэтому можно переопределить только нужный метод;
- фильтры действий запускаются после привязки модели, поэтому `context.ModelState` содержит ошибки, если привязка не удалась;
- При задании `context.Result` выполнение действия и более поздних фильтров будет отменено, однако более ранние фильтры выполнятся, как обычно.

Теперь реализуем `EnsureRecipeExistsAttribute`:
```csharp
public class EnsureRecipeExistsAttribute : ActionFilterAttribute
{
	public override void OnActionExecuting(ActionExecutingContext context)
	{
		var service = (RecipeService) context.HttpContext.RequestServices.GetService(typeof(RecipeService));
		var recipeId = (int) context.ActionArguments["id"];
		if (!service.DoesRecipeExist(recipeId))
		{
			context.Result = new NotFoundResult();
		}
	}
}
```

Здесь мы получаем экземпляр `RecipeService`[^2]  и значение параметра `id`, после чего проверяем, существует ли рецепт.

## Особый случай для фильтров действий
При использовании базового класса `ControllerBase` для контроллера, можно переопределить методы `OnActionExecuting` и `OnActionExecuted` этого класса; они будут запускаться на этапе фильтра действий для каждого действия контроллера. Метод `OnActionExecuting` будет выполняться перед всеми другими фильтрами действий, а `OnActionExecuted` - после.

[^1]: [[ch-13-creating-a-simple-filter|здесь]] сказано, что нельзя реализовывать и синхронную и асинхронную версию фильтра, и это по-прежнему верно; в [документации](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.actionfilterattribute?view=aspnetcore-5.0) сказано, что можно переопределить либо один или оба синхронных метода, либо асинхронный, но нельзя переопределять асинхронный и синхронные методы одновременно.

[^2]: [[ch-13-dependency-injection-with-filter-attributes|Подробнее о внедрении зависимостей в фильтрах]]