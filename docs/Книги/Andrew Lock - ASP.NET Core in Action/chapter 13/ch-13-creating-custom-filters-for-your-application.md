---
share: true
tags:
 - NET/ASPNETCore/filters
---
# Создание фильтров для приложения
Разберем каждый из [[ch-13-creating-a-simple-filter|шести типов фильтров]].
Для этого будем рефакторить веб-апи контроллер для [[ch-12-sample-application|приложения из предыдущей главы]]. Вот его начальный код:
```csharp
[Route("api/recipe")]
public class RecipeApiController : ControllerBase
{
	private const bool IsEnabled = true;
	public RecipeService _service;
	public RecipeApiController(RecipeService service)
	{
		_service = service;
	}
	
	[HttpGet("{id}")]
	public IActionResult Get(int id)
	{
		if (!IsEnabled) { return BadRequest(); }
		try
		{
			if (!_service.DoesRecipeExist(id))
			{
				return NotFound();
			}
			var detail = _service.GetRecipeDetail(id);
			Response.GetTypedHeaders().LastModified = detail.LastModified;
			return Ok(detail);
		}
		catch (Exception ex)
		{
			return GetErrorResponse(ex);
		}
	}
	
	[HttpPost("{id}")]
	public IActionResult Edit(int id, [FromBody] UpdateRecipeCommand command)
	{
		if (!IsEnbabled) { return BadRequest(); }
		try
		{
			if (!ModelState.IsValid)
			{
				return BadRequest(ModelState);
			}
			if (!_service.DoesRecipeExist(id))
			{
				return NotFound();
			}
			_service.UpdateRecipe(command);
			return Ok();
		}
		catch(Exception ex)
		{
			return GetErrorResponse(ex);
		}
	}
	
	private static IActionResult GetErrorResponse(Exception ex)
	{
		var error = new ProblemDetails
		{
			Title = "An error occured",
			Detail = ex.Message,
			Status = 500,
			Type = "https://httpstatuses.com/500"
		};
		return new ObjectResult(error) { StatusCode = 500 };
	}
}
```
Как видно, здесь много дублирующегося кода, а также кода, скрывающего смысл каждого действия.
#### [[ch-13-authorization-filters|Фильтры авторизации: защита API]]
#### [[ch-13-resource-filters|Фильтры ресурсов: прерывание выполнения методов действий]]
#### [[ch-13-action-filters|Фильтры действий: настройка привязки модели и результатов действий]]
#### [[ch-13-exception-filters|Фильтры исключений: собственная обработка исключений для методов действий]]
#### [[ch-13-result-filters|Фильтры результатов: настройка результатов действий перед их выполнением]]
Есть еще один тип фильтров, применяющийся только к Razor Pages
#### [[ch-13-page-filters|Фильтры страниц: настройка привязки модели для Razor Pages]]