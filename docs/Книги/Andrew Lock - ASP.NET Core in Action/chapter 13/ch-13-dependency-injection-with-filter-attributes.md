---
share: true
tags:
 - NET/ASPNETCore/filters
---
# Использование внедрения зависимостей с атрибутами фильтра
Атрибуты C# не позволяют передавать зависимости в свои конструкторы (кроме постоянных значений), что усложняет передачу зависимостей в фильтры, реализуемые как атрибуты.
Одним из способов обхода этого ограничения является использование локатора сервисов (Service Locator), а именно:
```csharp
var service = (RecipeService) context.HttpContext.RequestServices.GetService(typeof(RecipeService));
```
Однако, в мире внедрения зависимостей Service Locator считается антипаттерном.

Есть другой способ. Ключевой момент здесь — разделение фильтра на две части, а именно — класс фильтра, содержащий функциональность и атрибут, сообщающий фреймворку где и когда использовать фильтр.
```csharp
public class EnsureRecipeExistsFilter : IActionFilter //Не наследует Attribute
{
	private readonly RecipeService _service;
	public EnsureRecipeExistsFilter(RecipeService service)
	{
		_service = service;
	}
	
	public void OnActionExecuting(ActionExecutingContext context)
	{
		var recipeId = (int) context.ActionArguments["id"];
		if (!_service.DoesRecipeExist(recipeId))
		{
			context.Result = new NotFoundResult();
		}
	}
	
	public void OnActionExecuted(ActionExecutedContext context) { }
}

public class EnsureRecipeExistsAttribute : TypeFilterAttribute
{
	public EnsureRecipeExistsAttribute:base(typeof(EnsureRecipeExistsFilter)) {}
}
```
`TypeFilterAttribute` действует как фабрика для фильтра `EnsureRecipeExistsFilter` путём реализации интерфейса `IFilterFactory`. Когда вызывается действие, декорированное атрибутом `[EnsureRecipeExists]`, фреймворк вызывает фабричный метод `CreateInstance()` для атрибута, создавая новый экземпляр `EnsureRecipeExistsFilter`, используя контейнер внедрения зависимостей.

По умолчанию такую функциональность предоставляют два аналогичных класса с немного разным поведением:
- `TypeFilterAttribute` — загружает все зависимости фильтра из контейнера внедрения зависимостей и использует их для создания нового экземпляра фильтра;
- `ServiceFilterAttribute` — загружает *сам* фильтр из контейнера. При этом необходимо явно зарегистрировать фильтр в контейнере в `ConfigureServices`.
